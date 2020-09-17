## 每月每日一题接口

##### 请求url
    https://leetcode-cn.com/graphql
##### 请求方式
    POST
##### 请求头
    cookie需要LEETCODE_SESSION
##### 请求体
```json
{
    "operationName":"dailyQuestionRecords",
    "variables":{
      "year":2020,
      "month":8
    },
    "query":"query dailyQuestionRecords($year: Int!, $month: Int!) {\n  dailyQuestionRecords(year: $year, month: $month) {\n    date\n    question {\n      questionId\n      questionFrontendId\n      questionTitle\n      questionTitleSlug\n      translatedTitle\n      __typename\n    }\n    userStatus\n    __typename\n  }\n}\n"
}
```
##### 返回数据

```json
{
    "data": {
        "dailyQuestionRecords": [
            {
                "date": "2020-08-02",
                "question": {
                    "questionId": "114",
                    "questionFrontendId": "114",
                    "questionTitle": "Flatten Binary Tree to Linked List",
                    "questionTitleSlug": "flatten-binary-tree-to-linked-list",
                    "translatedTitle": "二叉树展开为链表",
                    "__typename": "QuestionNode"
                },
                "userStatus": "NOT_START",
                "__typename": "DailyQuestionNode"
            },
            {
                "date": "2020-08-01",
                "question": {
                    "questionId": "632",
                    "questionFrontendId": "632",
                    "questionTitle": "Smallest Range Covering Elements from K Lists",
                    "questionTitleSlug": "smallest-range-covering-elements-from-k-lists",
                    "translatedTitle": "最小区间",
                    "__typename": "QuestionNode"
                },
                "userStatus": "NOT_START",
                "__typename": "DailyQuestionNode"
            }
        ]
    }
}
```

## 根据题名获取题目信息接口


##### 请求url
    https://leetcode-cn.com/graphql
##### 请求方式
    POST
##### 请求头
    cookie需要csrftoken
    csrftoken=kuMviUEFlO5nr54j46DZCTKMyE6dyrPBHjzuR5Hj6dbh3jcH4LCKFFPMGNg0b3oA;
##### 请求体
```json
{
    "operationName":"questionData",
    "variables":{
      "titleSlug":"add-two-numbers"
      },
    "query":"query questionData($titleSlug: String!) {\n  question(titleSlug: $titleSlug) {\n    questionId\n    questionFrontendId\n    boundTopicId\n    title\n    titleSlug\n    content\n    translatedTitle\n    translatedContent\n    difficulty\n     topicTags {\n      name\n      slug\n      translatedName\n   }\n     codeSnippets {\n      lang\n      langSlug\n      code\n   }\n    stats\n      }\n}\n"
}
```
##### 返回数据
```json
{
    "data": {
        "question": {
            "questionId": "2",
            "questionFrontendId": "2",
            "boundTopicId": 988,
            "title": "Add Two Numbers",
            "titleSlug": "add-two-numbers",
            "content": "<p>You are given two <b>non-empty</b> linked lists representing two non-negative integers. The digits are stored in <b>reverse order</b> and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.</p>\r\n\r\n<p>You may assume the two numbers do not contain any leading zero, except the number 0 itself.</p>\r\n\r\n<p><b>Example:</b></p>\r\n\r\n<pre>\r\n<b>Input:</b> (2 -&gt; 4 -&gt; 3) + (5 -&gt; 6 -&gt; 4)\r\n<b>Output:</b> 7 -&gt; 0 -&gt; 8\r\n<b>Explanation:</b> 342 + 465 = 807.\r\n</pre>\r\n",
            "translatedTitle": "两数相加",
            "translatedContent": "<p>给出两个&nbsp;<strong>非空</strong> 的链表用来表示两个非负的整数。其中，它们各自的位数是按照&nbsp;<strong>逆序</strong>&nbsp;的方式存储的，并且它们的每个节点只能存储&nbsp;<strong>一位</strong>&nbsp;数字。</p>\n\n<p>如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。</p>\n\n<p>您可以假设除了数字 0 之外，这两个数都不会以 0&nbsp;开头。</p>\n\n<p><strong>示例：</strong></p>\n\n<pre><strong>输入：</strong>(2 -&gt; 4 -&gt; 3) + (5 -&gt; 6 -&gt; 4)\n<strong>输出：</strong>7 -&gt; 0 -&gt; 8\n<strong>原因：</strong>342 + 465 = 807\n</pre>\n",
            "difficulty": "Medium",
            "topicTags": [
                {
                    "name": "Linked List",
                    "slug": "linked-list",
                    "translatedName": "链表"
                },
                {
                    "name": "Math",
                    "slug": "math",
                    "translatedName": "数学"
                }
            ],
            "codeSnippets": [
                {
                    "lang": "C++",
                    "langSlug": "cpp",
                    "code": "/**\n * Definition for singly-linked list.\n * struct ListNode {\n *     int val;\n *     ListNode *next;\n *     ListNode(int x) : val(x), next(NULL) {}\n * };\n */\nclass Solution {\npublic:\n    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {\n        \n    }\n};"
                },
                {
                    "lang": "Java",
                    "langSlug": "java",
                    "code": "/**\n * Definition for singly-linked list.\n * public class ListNode {\n *     int val;\n *     ListNode next;\n *     ListNode(int x) { val = x; }\n * }\n */\nclass Solution {\n    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {\n\n    }\n}"
                },
                {
                    "lang": "Python",
                    "langSlug": "python",
                    "code": "# Definition for singly-linked list.\n# class ListNode(object):\n#     def __init__(self, x):\n#         self.val = x\n#         self.next = None\n\nclass Solution(object):\n    def addTwoNumbers(self, l1, l2):\n        \"\"\"\n        :type l1: ListNode\n        :type l2: ListNode\n        :rtype: ListNode\n        \"\"\""
                },
                {
                    "lang": "Python3",
                    "langSlug": "python3",
                    "code": "# Definition for singly-linked list.\n# class ListNode:\n#     def __init__(self, x):\n#         self.val = x\n#         self.next = None\n\nclass Solution:\n    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:"
                },
                {
                    "lang": "C",
                    "langSlug": "c",
                    "code": "/**\n * Definition for singly-linked list.\n * struct ListNode {\n *     int val;\n *     struct ListNode *next;\n * };\n */\n\n\nstruct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2){\n\n}\n\n"
                },
                {
                    "lang": "C#",
                    "langSlug": "csharp",
                    "code": "/**\n * Definition for singly-linked list.\n * public class ListNode {\n *     public int val;\n *     public ListNode next;\n *     public ListNode(int x) { val = x; }\n * }\n */\npublic class Solution {\n    public ListNode AddTwoNumbers(ListNode l1, ListNode l2) {\n\n    }\n}"
                },
                {
                    "lang": "JavaScript",
                    "langSlug": "javascript",
                    "code": "/**\n * Definition for singly-linked list.\n * function ListNode(val) {\n *     this.val = val;\n *     this.next = null;\n * }\n */\n/**\n * @param {ListNode} l1\n * @param {ListNode} l2\n * @return {ListNode}\n */\nvar addTwoNumbers = function(l1, l2) {\n\n};"
                },
                {
                    "lang": "Ruby",
                    "langSlug": "ruby",
                    "code": "# Definition for singly-linked list.\n# class ListNode\n#     attr_accessor :val, :next\n#     def initialize(val)\n#         @val = val\n#         @next = nil\n#     end\n# end\n\n# @param {ListNode} l1\n# @param {ListNode} l2\n# @return {ListNode}\ndef add_two_numbers(l1, l2)\n\nend"
                },
                {
                    "lang": "Swift",
                    "langSlug": "swift",
                    "code": "/**\n * Definition for singly-linked list.\n * public class ListNode {\n *     public var val: Int\n *     public var next: ListNode?\n *     public init(_ val: Int) {\n *         self.val = val\n *         self.next = nil\n *     }\n * }\n */\nclass Solution {\n    func addTwoNumbers(_ l1: ListNode?, _ l2: ListNode?) -> ListNode? {\n        \n    }\n}"
                },
                {
                    "lang": "Go",
                    "langSlug": "golang",
                    "code": "/**\n * Definition for singly-linked list.\n * type ListNode struct {\n *     Val int\n *     Next *ListNode\n * }\n */\nfunc addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {\n\n}"
                },
                {
                    "lang": "Scala",
                    "langSlug": "scala",
                    "code": "/**\n * Definition for singly-linked list.\n * class ListNode(var _x: Int = 0) {\n *   var next: ListNode = null\n *   var x: Int = _x\n * }\n */\nobject Solution {\n    def addTwoNumbers(l1: ListNode, l2: ListNode): ListNode = {\n\n    }\n}"
                },
                {
                    "lang": "Kotlin",
                    "langSlug": "kotlin",
                    "code": "/**\n * Example:\n * var li = ListNode(5)\n * var v = li.`val`\n * Definition for singly-linked list.\n * class ListNode(var `val`: Int) {\n *     var next: ListNode? = null\n * }\n */\nclass Solution {\n    fun addTwoNumbers(l1: ListNode?, l2: ListNode?): ListNode? {\n\n    }\n}"
                },
                {
                    "lang": "Rust",
                    "langSlug": "rust",
                    "code": "// Definition for singly-linked list.\n// #[derive(PartialEq, Eq, Clone, Debug)]\n// pub struct ListNode {\n//   pub val: i32,\n//   pub next: Option<Box<ListNode>>\n// }\n// \n// impl ListNode {\n//   #[inline]\n//   fn new(val: i32) -> Self {\n//     ListNode {\n//       next: None,\n//       val\n//     }\n//   }\n// }\nimpl Solution {\n    pub fn add_two_numbers(l1: Option<Box<ListNode>>, l2: Option<Box<ListNode>>) -> Option<Box<ListNode>> {\n        \n    }\n}"
                },
                {
                    "lang": "PHP",
                    "langSlug": "php",
                    "code": "/**\n * Definition for a singly-linked list.\n * class ListNode {\n *     public $val = 0;\n *     public $next = null;\n *     function __construct($val) { $this->val = $val; }\n * }\n */\nclass Solution {\n\n    /**\n     * @param ListNode $l1\n     * @param ListNode $l2\n     * @return ListNode\n     */\n    function addTwoNumbers($l1, $l2) {\n        \n    }\n}"
                },
                {
                    "lang": "TypeScript",
                    "langSlug": "typescript",
                    "code": "/**\n * Definition for singly-linked list.\n * class ListNode {\n *     val: number\n *     next: ListNode | null\n *     constructor(val?: number, next?: ListNode | null) {\n *         this.val = (val===undefined ? 0 : val)\n *         this.next = (next===undefined ? null : next)\n *     }\n * }\n */\n\nfunction addTwoNumbers(l1: ListNode | null, l2: ListNode | null): ListNode | null {\n\n};"
                }
            ],
            "stats": "{\"totalAccepted\": \"499.6K\", \"totalSubmission\": \"1.3M\", \"totalAcceptedRaw\": 499630, \"totalSubmissionRaw\": 1317898, \"acRate\": \"37.9%\"}"
        }
    }
}
```


## 每日一题接口

##### 请求url
    https://leetcode-cn.com/graphql
##### 请求方式
    POST
##### 请求头
    无需额外数据
##### 请求体
```json
{
  "operationName":"questionOfToday",
  "variables":{},
  "query":"query questionOfToday {\n  todayRecord {\n    question {\n      questionFrontendId\n      questionTitleSlug\n      }\n   date\n  }\n}\n"
}
```
##### 返回数据

```json
{
  "data": {
    "todayRecord": [
      {
        "question": {
          "questionFrontendId": "114",
          "questionTitleSlug": "flatten-binary-tree-to-linked-list"
        },
        "date": "2020-08-02"
      }
    ]
  }
}
```