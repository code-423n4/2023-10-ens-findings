Any comments for the judge to contextualize your findings
1) Some improvement can be made by adding mapping(address => address) proxyAddresses;

Approach taken in evaluating the codebase
1) I wrote a test to test all cases, with a focus on delegateMulti().
2) Draw a diagram to show how delegates and delegatees are related; for example, users u1 and u2 can multi-delegate to addresses A, B, C, respectively. 

[https://github.com/chaduke3730/diagrams/blob/main/multidelegate.png](https://github.com/chaduke3730/diagrams/blob/main/multidelegate.png)

    Architecture recommendations
1) Maybe add a whitelist for the delegatees. 

    Codebase quality analysis
1) Some of the code can be improved by adding mapping(address => address) proxyAddresses so that it is easier to return the proxy address of a given delegate.


    Centralization risks
1)  No centralization risk

    Mechanism review
1) Split delegateMulti() into three functions to improve UX: transferDelegation(); reimburse(); and  createDelegation(), which correspond to three cases: transfer from an array of delegator proxies to another array of delegator proxies; reimburse the user from an array of delegator proxies; and create a new set of delegator proxies. 

2)  Some edges cases can be detected such as zero address check and zero amount check

    Systemic risks
1) Not aware of any system risk





### Time spent:
5 hours