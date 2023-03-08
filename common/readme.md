## CLang XMl编程
### 1.下载与安装libxml2
Libxml2是一个C语言的XML程序库，可以简单方便的提供对XML文档的各种操作，并且支持XPATH查询，以及部分的支持XSLT转换等功能。Libxml2的下载地址是http://xmlsoft.org/，完全版的库是开源的，并且带有例子程序和说明文档。最好将这个库先下载下来，因为这样可以查看其中的文档和例子。

由于我是在linux下用C语言进行开发的，所以我下载的是libxml2-2.6.20.tar.gz版本的源码包。

具体安装步骤：

1、解压：$tar zxvf libxml2-2.6.20.tar.gz

2、进入解压后的安装目录：$cd libxml2-2.6.20

3、安装三部曲： 

			  1）$./configure

        	  2)$make

              3)$make install

安装完毕。
### 2. Libxml2中的数据类型和函数
一个函数库中可能有几百种数据类型以及几千个函数，但是记住大师的话，90%的功能都是由30%的内容提供的。对于libxml2，我认为搞懂以下的数据类型和函数就足够了。

#### 2.1 内部字符类型xmlChar
xmlChar是Libxml2中的字符类型，库中所有字符、字符串都是基于这个数据类型。事实上它的定义是：xmlstring.h

typedef unsigned char xmlChar;

使用unsigned char作为内部字符格式是考虑到它能很好适应UTF-8编码，而UTF-8编码正是libxml2的内部编码，其它格式的编码要转换为这个编码才能在libxml2中使用。

还经常可以看到使用xmlChar*作为字符串类型，很多函数会返回一个动态分配内存的xmlChar*变量，使用这样的函数时记得要手动删除内存。

#### 2.2 xmlChar相关函数
如同标准c中的char类型一样，xmlChar也有动态内存分配、字符串操作等相关函数。例如xmlMalloc是动态分配内存的函数；xmlFree是配套的释放内存函数；xmlStrcmp是字符串比较函数等等。

基本上xmlChar字符串相关函数都在xmlstring.h中定义；而动态内存分配函数在xmlmemory.h中定义。

#### 2.3 xmlChar*与其它类型之间的转换
另外要注意，因为总是要在xmlChar*和char*之间进行类型转换，所以定义了一个宏BAD_CAST，其定义如下：xmlstring.h

\#define BAD_CAST (xmlChar *)

原则上来说，unsigned char和char之间进行强制类型转换是没有问题的。

#### 2.4 文档类型xmlDoc、指针xmlDocPtr
xmlDoc是一个struct，保存了一个xml的相关信息，例如文件名、文档类型、子节点等等；xmlDocPtr等于xmlDoc*，它搞成这个样子总让人以为是智能指针，其实不是，要手动删除的。

xmlNewDoc函数创建一个新的文档指针。

xmlParseFile函数以默认方式读入一个UTF-8格式的文档，并返回文档指针。

xmlReadFile函数读入一个带有某种编码的xml文档，并返回文档指针；细节见libxml2参考手册。

xmlFreeDoc释放文档指针。特别注意，当你调用xmlFreeDoc时，该文档所有包含的节点内存都被释放，所以一般来说不需要手动调用xmlFreeNode或者xmlFreeNodeList来释放动态分配的节点内存，除非你把该节点从文档中移除了。一般来说，一个文档中所有节点都应该动态分配，然后加入文档，最后调用xmlFreeDoc一次释放所有节点申请的动态内存，这也是为什么我们很少看见xmlNodeFree的原因。

xmlSaveFile将文档以默认方式存入一个文件。

xmlSaveFormatFileEnc可将文档以某种编码/格式存入一个文件中。

#### 2.5 节点类型xmlNode、指针xmlNodePtr 

	typedef struct _xmlNode xmlNode;

	typedef xmlNode *xmlNodePtr;

	struct _xmlNode {

		void           *_private; /* application data */

		xmlElementType   type;   /* type number, must be second ! */

		const xmlChar   *name;      /* the name of the node, or the entity */

		struct _xmlNode *children; /* parent->childs link */

		struct _xmlNode *last;   /* last child link */

		struct _xmlNode *parent;/* child->parent link */

		struct _xmlNode *next;   /* next sibling link */

		struct _xmlNode *prev;   /* previous sibling link */

		struct _xmlDoc *doc;/* the containing document */

		/* End of common part */

		xmlNs           *ns;        /* pointer to the associated namespace */

		xmlChar         *content;   /* the content */

		struct _xmlAttr *properties;/* properties list */

		xmlNs           *nsDef;     /* namespace definitions on this node */

		void            *psvi;/* for type/PSVI informations */

		unsigned short   line;   /* line number */

		unsigned short   extra; /* extra data for XPath/XSLT */

	};
可以看到，节点之间是以链表和树两种方式同时组织起来的，next和prev指针可以组成链表，而parent和children可以组织为树。同时还有以下重要元素：
节点中的文字内容：content；

节点所属文档：doc；

节点名字：name；

节点的namespace：ns；

节点属性列表：properties；

Xml文档的操作其根本原理就是在节点之间移动、查询节点的各项信息，并进行增加、删除、修改的操作。

xmlDocSetRootElement函数可以将一个节点设置为某个文档的根节点，这是将文档与节点连接起来的重要手段，当有了根结点以后，所有子节点就可以依次连接上根节点，从而组织成为一个xml树。

#### 2.6 节点集合类型xmlNodeSet、指针xmlNodeSetPtr
节点集合代表一个由节点组成的变量，节点集合只作为Xpath的查询结果而出现（XPATH的介绍见后面），因此被定义在xpath.h中，其定义如下：
	/*

	* A node-set (an unordered collection of nodes without duplicates).

	*/

	typedef struct _xmlNodeSet xmlNodeSet;

	typedef xmlNodeSet *xmlNodeSetPtr;

	struct _xmlNodeSet {
		int nodeNr;          /* number of nodes in the set */

		int nodeMax;      /* size of the array as allocated */

		xmlNodePtr *nodeTab;/* array of nodes in no particular order */

		/* @@ with_ns to check wether namespace nodes should be looked at @@ */

	};

可以看出，节点集合有三个成员，分别是节点集合的节点数、最大可容纳的节点数，以及节点数组头指针。对节点集合中各个节点的访问方式很简单，如下：

	xmlNodeSetPtr nodeset = XPATH查询结果;

	for (int i = 0; i < nodeset->nodeNr; i++)

	{
		nodeset->nodeTab[i];

	}
### 3. 简单xml操作例子
了解以上基本知识之后，就可以进行一些简单的xml操作了。当然，还没有涉及到内码转换（使得xml中可以处理中文）、xpath等较复杂的操作。

#### 3.1 创建xml文档
有了上面的基础，创建一个xml文档显得非常简单，其流程如下：

l 用xmlNewDoc函数创建一个文档指针doc；

l 用xmlNewNode函数创建一个节点指针root_node；

l 用xmlDocSetRootElement将root_node设置为doc的根结点；

l 给root_node添加一系列的子节点，并设置子节点的内容和属性；

l 用xmlSaveFile将xml文档存入文件；

l 用xmlFreeDoc函数关闭文档指针，并清除本文档中所有节点动态申请的内存。

注意，有多种方式可以添加子节点：第一是用xmlNewTextChild直接添加一个文本子节点；第二是先创建新节点，然后用xmlAddChild将新节点加入上层节点。
源代码文件是CreateXmlFile.cpp