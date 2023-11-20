# Solidity-bootcamp-4

###Task-4

In the previous lesson we have created a function to calculate the state of the proposal so that we can see if the proposal is succeeded or failed. We can have many different logics for the calculation other than the one we implemented. So, your task is:

1. To create your own logic and implement it instead of the one we created 
2. Submit your code.

Solution:

```sol
function calculateCurrentState() private view returns(bool) {
    Proposal storage proposal = proposal_history[_counter.current()];

    uint256 totalVotes = proposal.approve + proposal.reject + proposal.pass;

    // Calculate the approval rate as a percentage
    uint256 approvalRate = (proposal.approve * 100) / totalVotes;

    // Determine the success threshold based on the total votes
    uint256 successThreshold;
    if (totalVotes <= 10) {
        successThreshold = 60; // 60% approval required for proposals with 10 or fewer votes
    } else {
        successThreshold = 50; // 50% approval required for proposals with more than 10 votes
    }

    // Check if the approval rate meets the success threshold
    return approvalRate >= successThreshold;
}
```
