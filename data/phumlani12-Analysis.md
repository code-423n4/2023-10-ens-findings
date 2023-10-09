Analysis of the ENS Contract

Introduction
I have not seen any analysis examples before, so I wouldn't know the best way to write it, whether with subheadings or just as one long essay. I should also mention that this was my first-ever submission on any platform. The registration on the platform took a bit longer than I expected, but I finally got it done mid-day on Friday. Regarding the contract, I saw it mid-day on Friday, quickly glanced through it, and knew it was the "one". I waited for the anxiety of registering on the platform to die down, as I wanted a clear mind for the audit. What was amazing and felt like a coincidence is that I was watching a video about what ENS is on Friday morning, which confirmed that I needed to participate in this contest, despite it being my first time.

Auditing
With that said, I looked at the contract several times on Friday afternoon, went through the documentation, and tried to understand the protocol and what it was trying to achieve. I went to bed, and on Saturday morning, I started analyzing it. I did not use any tools; instead, I wrote my tests on Foundry. However, it wasn't as fruitful as I struggled to deploy it due to the address and metadata URL being prerequisites before deploying. I manually went through the contract and noticed the presence of dynamic arrays. I told myself that I didn't want low-hanging fruits; I wanted something more. I kept staring at it and then found something suspicious: the reimburse function! I wanted to run tests but couldn't do so because of the aforementioned reasons. I went through the code again. I appreciated the comments and how clean the code was, but I also identified a vulnerability. Balance checks are very important, especially before changing states. I even wrote a contract that I could use to exploit the contract, although I didn't actually execute it due to the reasons mentioned. It is also important that, just today I eventually decided to go ahead and report the potential DOS that comes with Dynamic arrays. Could have done better with my POC's. 

Code
I think it is written well, well formatted, with enough comments. It is all clear and concise in my personal opinion. There aren't any verbose functions, which makes it clean.

Security
It is not the most secure, which is why it is on the platform. The code can be made more secure after this contest. There are things that can be fixed and improved.

### Time spent:
15 hours