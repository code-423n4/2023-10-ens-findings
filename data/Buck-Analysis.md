Analysis *

I am new to auditing and this is my first submission for an audit

ENS - voting delegation

I skimmed the docs, since the scope of this audit is for delegation of voting, most of the documentation is far outside the scope.
Read through the code, then went to the provided test.  I went through the test to see if I could determine if anything was missing.

The tests did not check for side effects or check all of the amounts were correct, but did for most.  Noticed that a test was using 0
for the amounts, (using balanceOf and math to make the params), so the end result was correct because math works, but did not actually 
test the functionality because the amounts were zero.  This caused to look closer at this aspect, amounts of zero were passed and it the
smart contract was executed.  Showing that it was not necessary to have tokens in order to call the function and deploy and allocate 
proxy delegators. 

This is a simple codebase because of the limited scope, one of the functions _delegateMulti(....)  has 
```
if {
   ...
} elseif {
   ...
} elseif {
   ...
}
```
calling different functions in each block after setting variables that may go unused.
It does function well, I restructured the function and ran gas analysis, it saved a little gas but was not major enough to submit.

Except for the issue I submitted the codebase really is tight and succinct. 
The delegation process is owned by the delegator, the risk of centralization is not a risk from the technology, it is a risk of the community delegating to one, or a small group of addresses.

Mechanism review
The mechanism is straightforward. The way that the reimburse is implemented it can transfer dust (when changing from differing numbers of delegators e.g. 2 -> 3)
where there is dust left and is reimbursed, the soution for this can be done off-chain and I recommend testing for that.

Did not see any large systemic risks.

### Time spent:
4 hours