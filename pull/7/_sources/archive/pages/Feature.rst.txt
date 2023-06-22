| **Name**: }}
| {{#if:\| **{{#if:|Tickets|Ticket}}**:
  [https://pagure.io/freeipa/issue/ #] {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }} {{#if:\| ,
  [https://pagure.io/freeipa/issue/ #] }}
| }} {{#if:\| **Target version**:
  ` <Releases/%7B%7B%7Bversion%7D%7D%7D>`__
| \| }} {{#if:\| **Author**:
  ` <User:%7B%7B%7Bauthor%7D%7D%7D>`__\ {{#if:|,
  ` <User:%7B%7B%7Bauthor2%7D%7D%7D>`__}}{{#if:|,
  ` <User:%7B%7B%7Bauthor3%7D%7D%7D>`__}}{{#if:|,
  ` <User:%7B%7B%7Bauthor4%7D%7D%7D>`__}}
| }} {{#if:\| **Green_tick.png Reviewed by**:
  ` <User:%7B%7B%7Breviewer%7D%7D%7D>`__ {{#if:|,
  ` <User:%7B%7B%7Breviewer2%7D%7D%7D>`__}}{{#if:|,
  ` <User:%7B%7B%7Breviewer3%7D%7D%7D>`__}}{{#if:|,
  ` <User:%7B%7B%7Breviewer4%7D%7D%7D>`__}}
| \| **Incomplete.png Pending review**
| }} {{#if:\| **Test plan**:
  `{{{test_plan}}} <%7B%7B%7Btest_plan%7D%7D%7D>`__
| }} **Last updated**: -- by ` <User:%7B%7BREVISIONUSER%7D%7D>`__

`Category:FreeIPA {{{version}}}
Design <Category:FreeIPA_%7B%7B%7Bversion%7D%7D%7D_Design>`__
`Category:FreeIPA Design Proposal <Category:FreeIPA_Design_Proposal>`__
