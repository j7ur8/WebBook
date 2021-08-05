# PHP的编译与执行

​		PHP是一门解释型语言，php code直接被PHP的解释器解析后执行对应操作，不会被翻译成机器语言然后再去执行操作。PHP的核心ZendVM（虚拟机）就是PHP的解释器，也正是ZendVM也赋予了php code多平台运行的能力。想要了解php code是如何被正确运行，就要了解ZendVM的工作流程，

​		虚拟机大多都是模拟真实机器处理过程，语法逻辑结构上大多都大同小异，不同的是对运算符、数据类型的定义。所以在探究一个虚拟机的内部结构时，我们需要有明确的目标

- 虚拟机内部用来描述指令执行过程的指令集
- 单个指令对应的解释过程

## 概述

​		ZendVM有编译和执行两个模块。编译过程将php code编译为ZendVM内部定义好的指令集。执行过程将一条条指令调用到执行器中去执行。

- 单条指令在php中被称为"opline"
- `jmp 10000`中的关键字jmp在php中被称为"opcode"
- opline可以有两个操作数op1和op2，也可以只有一个操作数op1，也可以没有操作数，但至多有两个操作数。

### _zend_op

​		Opline结构如下：

```c
struct _zend_op {
    const void *handler;
    znode_op op1;  # 操作数op1的定义
    znode_op op2;  # 操作数op2的定义
    znode_op result; #见下文
    uint32_t extended_value;
    uint32_t lineno;
    zend_uchar opcode;
    zend_uchar op1_type;
    zend_uchar op2_type;
    zend_uchar result_type;
};
 
typedef struct _zend_op zend_op;
```

​		zend_unchar定义操作数的类型，可以分为下面5种

```c
#define IS_UNUSED    0        /* Unused operand */
#define IS_CONST    (1<<0)
#define IS_TMP_VAR    (1<<1)
#define IS_VAR        (1<<2)
#define IS_CV        (1<<3)    /* Compiled variable */
```

- IS_UNUSED表示操作数未使用
- IS_CONST表示操作数类型是常量
- IS_TMP_VAR为临时变量，是一种中间变量，出现在复杂表达式计算的时候，比如进行字符串拼接（双常量字符串拼接时无临时变量）
- IS_VAR是PHP内的变量，大多数情况下表示opline的返回值，但在php code种并未显示表现出来，如if(random()){}的random()返回值就是VAR变量类型
- IS_CV是php code里显示定义的变量，如$a等

​		znode_op是操作数的内容结构，如下

```c
typedef union _znode_op {
    uint32_t      constant;
    uint32_t      var;
    uint32_t      num;
    uint32_t      opline_num; /*  Needs to be signed */
#if ZEND_USE_ABS_JMP_ADDR
    zend_op       *jmp_addr;
#else
    uint32_t      jmp_offset;
#endif
#if ZEND_USE_ABS_CONST_ADDR
    zval          *zv;
#endif
} znode_op;
```

​		znode_op是一个union结构，分为相对寻址和绝对寻址两种情况。而opline的操作数是去哪儿寻址呢？这就需要了解zend_op_array和zend_execute_data。

### zend_op_array

```c
struct _zend_op_array {
    /* Common elements */
    zend_uchar type;
    zend_uchar arg_flags[3]; /* bitset of arg_info.pass_by_reference */
    uint32_t fn_flags;
    zend_string *function_name;
    zend_class_entry *scope;
    zend_function *prototype;
    uint32_t num_args;
    uint32_t required_num_args;
    zend_arg_info *arg_info;
    /* END of common elements */
 
    int cache_size;     /* number of run_time_cache_slots * sizeof(void*) */
    int last_var;       /* number of CV variables */
    uint32_t T;         /* number of temporary variables */
    uint32_t last;      /* number of opcodes */
 
    zend_op *opcodes;
    ZEND_MAP_PTR_DEF(void **, run_time_cache);
    ZEND_MAP_PTR_DEF(HashTable *, static_variables_ptr);
    HashTable *static_variables;
    zend_string **vars; /* names of CV variables */
 
    uint32_t *refcount;
 
    int last_live_range;
    int last_try_catch;
    zend_live_range *live_range;
    zend_try_catch_element *try_catch_array;
 
    zend_string *filename;
    uint32_t line_start;
    uint32_t line_end;
    zend_string *doc_comment;
 
    int last_literal;
    zval *literals;
 
    void *reserved[ZEND_MAX_RESERVED_RESOURCES];
};
```

​		zend_op_array不仅包含编译过程在内所产生的所有opline集合，还包含编译过程中动态生成的关键数据，先介绍一下几种：

- vars是CV变量名组成的指针数组，相当于一张CV变量名组成的表，是不存在重复变量名的，对应的变量值存储在另外一个结构上
- last_var表示最后一个CV变量的序号，也可以代表CV变量的数量

- literals是存储编译过程产生的常量数组，根据编译过程中依次出现的顺序存放在改数组中
- last_literal表示当前存储的常量的数量
- T表示TMP_VAR和VAR的数量

​		以上就是操作数部分信息存储的地方。可以看到zend_op_array仅分配了CV变量名数组，但并没有存储CV变量值；TMP_VAR和VAR变量亦是如此，只有简单的数量记录。

### zend_execute_data

```c
struct _zend_execute_data {
    const zend_op       *opline;           /* executed opline                */
    zend_execute_data   *call;             /* current call                   */
    zval                *return_value;
    zend_function       *func;             /* executed function              */
    zval                 This;             /* this + call_info + num_args    */
    zend_execute_data   *prev_execute_data;
    zend_array          *symbol_table;
#if ZEND_EX_USE_RUN_TIME_CACHE
    void               **run_time_cache;   /* cache op_array->run_time_cache */
#endif
};
```

​		zend_execute_data相当于执行编译oplines的contenxt（上下文），是通过具体的某个zend_op_array的结构信息初始化产生的，所以一个zend_execute_data对应一个zend_op_array，用来存储解释运行过程中产生的局部比那里、当前执行的opline、上下文直接调用的关系、调用者信息、符号表等。

​		CV变量、TMP_VAR、VAR变量也正是分配在这个结构上，并且还是动态分配紧接zend_execute_data结构后面，首先是分配CV变量，然后依次分配VAR、TMP
_VAR变量。

### 取zend_execute_data

​		关于如何在该动态局部变量区取值，需要注意网上基本都是使用如下命令

```c
(zval *)(((char *)(execute_data))+96)
```

去取第一个值，但是需要注意：

- sizeof(zend_execute_data)在不同的php版本中，其值不一定是96。动态分配的变量在zend_execute_data结构的末尾，所以你需要提前知道这个结构的大小。

- zend_execute_data分配的大小是按照sizeof(zval)的整数倍来分配的，即16对齐。

 ```c
  #define ZEND_CALL_FRAME_SLOT \
      ((int)((ZEND_MM_ALIGNED_SIZE(sizeof(zend_execute_data)) + ZEND_MM_ALIGNED_SIZE(sizeof(zval)) - 1) / ZEND_MM_ALIGNED_SIZE(sizeof(zval))))
   
  static zend_always_inline uint32_t zend_vm_calc_used_stack(uint32_t num_args, zend_function *func)
  {
      uint32_t used_stack = ZEND_CALL_FRAME_SLOT + num_args;
   
      if (EXPECTED(ZEND_USER_CODE(func->type))) {
          used_stack += func->op_array.last_var + func->op_array.T - MIN(func->op_array.num_args, num_args);
      }
      return used_stack * sizeof(zval);
  }
 ```

综上所述，大概明白了CV变量、TMP_VAR变量、VAR变量的存储位置，我们就可以获取opline中的操作数

- 通过znode_op.var、znode_op.constand进行相对寻址。var代表CV、TMP_VAR、VAR的相对位置。如0x50、0x60、0x70这样相对于zend_exe
  cute_data结构起始地址的位置
- 使用zval *指针进行直接寻址
- 用jmp进行直接跳转或间接跳转

关于opline（_zend_op）结构中的handler字段会在后面详细介绍。本节主要对常用结构（zend_op、znode、opcode_array、execute_data）进行简单介绍。下面就开始具体介绍实现过程的细节以及结构具体应用在哪些地方。

## 编译过程
