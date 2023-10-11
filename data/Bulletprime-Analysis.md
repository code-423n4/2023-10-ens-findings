Overview
The Ethereum Name Service (ENS) is an open-ended and decentralized versatile naming system with unique feature based on the Ethereum blockchain. This protocol deals with the ERC20votes as an extension of the ERC20token that allow users to utilize their tokens as voting power. The multi delegate contract uses the ERC1155 and a proxy contract, the multi delegate creates a proxy contract that receives the users tokens and delegate them to the target address. In exchange the user receives an ERC1155 token that tracks the ownership of the ERC20 vote tokens inside the proxy.

Audit Approach
I paid attention to my successful steps that has proven to fetch me low hanging fruits.
1.Read the Audit Readme.md and took necessary notes.
2.Retentively had a visual idea of the multi delegate contracts intent.
3.Gathered knowledge on proxy contracts and their proper implementations in addition to the ERC20 and ERC1155 token functionality and limitations.
4.Although no much documentation on this particular project, much time was spent reading about similar projects and how the ENS protocol can be better.
5.Analyzed the over all codebase one iteration at a time.
6.Then I read old audits and already known findings on similar projects, taking notes of things that could go wrong, the same time going through the bot races findings.
7.Then setup my testing environment, Run the tests to checks all test passed. I used npx hard-hat node to test the protocol. 
8.Finally, I started with auditing the code base in-depth, that way I started understanding line by line code and took the necessary notes to ask some questions.

THE AUDIT PROCESS
The first stage of the audit
During the initial stage of the audit for the ENS Protocol, the primary focus was on analyzing gas usage and quality assurance (QA) aspects. This phase of the audit aimed to ensure the efficiency of gas consumption. Although most of my gas analysis and QA were already recorded by bots, so I didn't bother including them to my list of findings.
The second stage of the audit process.
The focus shifted towards understanding the protocol usage in more detail. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components, what the codes is aimed to gain and if it corelates to the protocol’s functionality.
The third stage 
the focus was on thoroughly examining and marking out any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive vulnerability assessments and identifying potential weaknesses in the system both through testing and manual scanning.
The fourth stage 
Comprises of a comprehensive analysis and testing of the previously identified doubtful and vulnerable areas were conducted. This stage involved diving deeper into these areas, performing in-depth examinations, and subjecting them to rigorous testing, Although my suspected bugs and were actually false positives as in-depth research proved the codes were actually accurate.

LEARNINGS
Although I personally didn't find any critical issue, I believe the codebase being simple, direct and less complicated helped reduced critical issues and developers should take advantage of this, but that should not go without prioritizing security, as there were still vulnerabilities bots were able to spot conducting thorough risk assessments, and actively involving the community as this can further strengthen the protocols position as the lead Naming service system.

Architecture Recommendation
The protocol supports a very generic implementation and proxy inheritance and as the team understands, this leads to a broad variety of malicious deployed well risks. 
1.It should be a high priority task for the team to find good ways of representing all Well trust assumptions to its users, recognize that ENS relies on smart contract and owners of domain can be external addresses or smart contracts themselves. Although there is an automatic check for the network or address you are interacting with. Which I see as a good model.
2.Be aware of the capabilities of delegators within the ENS Protocol, managing ownership and access control is crucial. bot-report.mdhttps://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#l01-use-ownable2step-instead-of-ownable.
3.There should always be an eye out for proxy standardization, functionalities and limitations, stay updated on new developments.
4.Security and best practices, prioritizing security, following best practices for managing keys, votes, access control and contract interaction can help maintain the protocols standards.
5. There was no proper Documentation to this particular project which I considered to be a setback and limiting the community of valuable insights and understanding of the protocol, I believe adequate documentation should be provided as the Ethereum ecosystem is continuously evolving, and community support can be invaluable.

Qualitative code analysis
Code quality is very mature, straightforward and simple to understand. The test suite is pretty comprehensive and fuzz tests are a great way to complement the static tests. My suggestion is to add integration tests to verify behavior of the system as a whole, rather than all its specific sub-components. There were also vulnerable package versions used.
https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#l04-vulnerable-versions-of-packages-are-being-used
Add inline comments for complex logic (source, target) as encountered to make code more understandable. Also good practice to add detailed information in the events logs involved in certain actions.

Centralization risks 
A single point of failure is not acceptable for this project, centrality risk is found for privileged function, Contracts with privileged functions need owner/admin to be trusted not to perform malicious updates or drain funds. https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#m01-centralization-risk-for-privileged-functions

Mechanism review
run in first terminal
$ npx hardhat node
_run in another terminal
yarn test test/delegatemulti.js
_for coverage
yarn coverage
The test coverage which covered only 80%, showed that the protocol may have undiscovered vulnerabilities, and adequate security and manual audits is best put in place to maintain standards and combat unnoticed critical issues and reduce risks of attacks.

Systemic review
The ERC20MultiDelegate proxy contract is prone to vulnerabilities as there are often used to implement upgradable logic contract, while this proxy contract offers flexibility, they can also introduce vulnerabilities. The team should bear in mind logic flaws, ownership issues etc. commonly associated with proxy contracts. https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#l01-use-ownable2step-instead-of-ownable
The team should also prepare multiple recovery scenarios and set up adequate action channels to prepare and validate function calls and input parameters, and carefully manage ownership of the contract. while security checks is a continuous process, there should be regular code review that can help identify and fix potential vulnerabilities.

Non-functional aspects
Aim for high test coverage to validate the contract’s behavior and catch potential bugs or vulnerabilities.
The protocol lacks adequate documentation leaving most auditors with a poor understanding of the intended contract.
Its best to explain over all protocol, with architecture and documentations, protocols are easier to understand.

### Time spent:
19 hours