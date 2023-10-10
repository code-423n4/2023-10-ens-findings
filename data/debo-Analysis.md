## Any comments for the judge to contextualise your findings
I found access control and invalid validation issues.

## Approach taken in evaluating the codebase
Search for keywords that are vulnerabilities and the write impact assessment and PoC.

## Architecture recommendations
Gas Optimisation: The delegateMulti function increments over lots in an array.

## Codebase quality analysis
Efficiency: The delegateMulti function increments over lots in an array. This is expensive.

## Centralisation risks
Lack of Decentralization in Decision Making: Decisions are made by the owner only, which is a risk to users.

## Mechanism review
There is a complicated delegate mechanism in place.

## Systemic risks
Gas is costly.
All governance decisions are made by the owner of the contract.

### Time spent:
35 hours