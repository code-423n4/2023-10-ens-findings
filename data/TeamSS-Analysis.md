### Any comments for the judge to contextualize your findings
Context was added where it was relevant to aid the judges. 

###  Approach taken in evaluating the protocol

Since this audit was focused on the ENS MultiDelegate codebase, below were the steps taken: 

1. Since the docomentation for the `ERC20MultiDelegate.sol` contract is not available on the current version of the [ENS docs](https://docs.ens.domains/), We immediately dived into the contract codebase. We learnt about the following components used in the codebase. 
        1. ERC20Votes 
        2. ERC1155 
        3. Proxy-based delegation. 
2.  Drew one diagram based on this understanding. [[ENS MutliDegelate](https://postimg.cc/QVgBxxQy)]
3.  Started to explore the codebase. 
4. When in doubt or confused, We ask in the discord channel dedicated or at one point a discussion with [some wardens on X](https://twitter.com/0xSimeon/status/1711423484725780619?t=xNsyvboXGd07D_TqivctEA&s=19) for the audit. The sponsor and other wardens were kind enough to answer our questions.  
5.  Next, We evaluate the contracts's logic, following the chained internal functions since there was only one function that can be called externally. 
6.  Over the course of the above processes, we took notes. Our primary objective was to break some of the assumptions and invariants of the system like `assert(amount <= balance);`. 

### Observations 
- The ERC20MultiDelegate contract was a hard nut to crack. 
- The codebase was easy to get by and understand but could use more Natspec documentation and comments so most of the nitty gritty details of the functions can be inferred. 


### Architecture recommendations
- N/A



### Systemic and/or Centralization risks
None of the core functions have centralization risks. open and permissionless. 


### Time spent:
28 hours.




### Time spent:
28 hours