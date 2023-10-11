# Auditing of ENS contract
ENS has once again applied for a Code4rena audit and this time with a focus on a multi-delegation mechanism for ERC20 tokens. This audit involves one smart contract with 216 SLOC and a 33,050 USDC rewards. My first thoughts about this competition are that it is good for me and short enough to really focus on more complex logic vulnerabilities. I have previously found a unique medium severity vulnerability in a competition and this is a great spot to find another one! Since the price pool and scope is fairly small there needs to be a bigger focus on Low/QA report as well as this analysis. 

My goals are to get more familiar with the ENS protocol and come back to auditing after an intense 10 month break from web3. This break was due to my duties to serve my country.

## Reading documentation
Starting the contest is usually the most interesting time for me. I enjoy reading documentation of new protocols and smart contract innovations. I read the whole ENS documentation for context and then started to dig deeper in multi-delegation mechanisms.
ENS governance documentation is relevant to voting mechanism, so reading it was my first priority.
There wasn't a lot of documentation regarding the ```ERC20MultiDelegate.sol``` contract but I got a good grip on it by reading through the test code. The tests described how the protocol should be used and when it would not work. It was surprisingly easy to read the tests as if they were written documentation.

## Auditing
I spent all together 20 hours of auditing for this contest. The first 4 hours was used to get a good understanding of the codebase and how the flow of delegating through proxies worked. I am surprised how simple the code looked at the start but then how specifically every aspect was crafted. This reminded me of the importance of designing protocols well. It was clear that the developer(s) used a substantial time planning all functionalities of this contract before actually coding it. 

Having audited all of the code for about 12 hours I started to see some possible vulnerabilities. I went down several rabbit holes but didn't find anything substantial. Reading the Discord chat kept me motivated since there was a lot of good and positive conversation about this contest and the code (much more than I had seen before). One of the reason for this could be that the sponsor was not available for the first two to three days of the contest which made everyone get together and chat it up. 

I found a possible bug and made a medium submission from it after writing the report. Reporting something that is on the fence of medium and low I usually decide to submit it as medium since there is always a chance and if it is ends up being low it will be downgraded anyways. 

The architect of this contract is very good and it was quite hard to find anything worth submitting. It is always exciting to see the submissions through backstage access and learn from missed vulnerabilities.

## End note

I enjoyed this audit because it was fairly short but still included some advanced solidity architecture. Having learned from my mistakes I think that attending this audit was a good decision and the codebase wasn't too hard for me. Thank you ENS team and Code4rena team!  

### Time spent:
20 hours