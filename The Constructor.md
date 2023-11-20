So far, we added the Proposal struct, a mapping and a function to create Proposal. Currently anyone can create a Proposal in our contract. What we want is that only the owner of this contract can create new proposals.


To achieve that, first we need to add a new line to the beginning of our contract.


address owner;


We created a new variable named owner which will be representing the owner of the contract.


Now that we have the necessary variable for the owner, we need set the owner.


The best place to set the owner of the contract inside the constructor.


Constructor runs once when we first deploy our contract. Here we want to set owner of the contrat to the address that has deployed the contract.


We can have the address that made the transaction with: msg.sender

```
constructor() {
    owner = msg.sender;
}
```

Now we have the owner but how can we use this owner in the function so that the only owner of the contract can use the function?


---

### OnlyOwner Modifier

In the last lesson, we created a new variable named owner to represent the owner of the contract and we set the owner to the msg.sender in our contructor. Now, we want to use this owner in our create function so that only the owner of the contract can use this function.


We can achieve this with a modifier.


- Modifiers are defined using the modifier keyword followed by the modifier name and parameters if any.


- Within the modifier, custom logic can be added before and/or after the _; statement.


- By default, modifiers will run the logic before executing the function body. The _; tells it to then execute the function.


- To use a modifier on a function, include it after the function name followed by parameters.


Let's create our onlyOwner modifier.

```
modifier onlyOwner() {
    require(msg.sender == owner);
    _;
}
```

- modifier onlyOwner() {...} - This starts the definition of the modifier called onlyOwner.


- require(msg.sender == owner) - This line checks that the msg.sender, the account calling the function, is equal to the stored owner address.


- _; - This tells Solidity to execute the rest of the function code after completing the modifier logic.


To use the onlyOwner modifier, we just need to add the onlyOwner keyword at the end of the function definition.


Here is the modified version of our function:

```
function create(string calldata _description, uint256 _total_vote_to_end)
external onlyOwner {
    _counter.increment();
    proposal_history[_counter.current()] = Proposal(_description, 0, 0, 0, _total_vote_to_end, false, true);
}
```

Now our function can only be called by the owner of this contract.


Finally, let's add a function to set the owner of the contract since we may need to transfer ownership of the contract in the future.

```
function setOwner(address new_owner) external onlyOwner {
    owner = new_owner;
}
```

This function will transfer the ownership of the contract to the given address as the parameter and it can only be called by the owner of the contract.

---

### Voting Functionality

In this part, we will be implementing an execute function to vote on a proposal.


Let's start by defining our function:


function vote(uint8 choice) external {
    // Function logic
}

We will be using uint8 to represent the three different options we can have:


 Approve -> will be represented by 1. 
 Reject -> will be represented by 2. 
 Pass -> will be represented by 0. 

Let's move on with the function logic.


// First part
Proposal storage proposal = proposal_history[_counter.current()];
uint256 total_vote = proposal.approve + proposal.reject + proposal.pass;

// Second part
if (choice == 1) {
    proposal.approve += 1;
    proposal.current_state = calculateCurrentState();
} else if (choice == 2) {
    proposal.reject += 1;
    proposal.current_state = calculateCurrentState();
} else if (choice == 0) {
    proposal.pass += 1;
    proposal.current_state = calculateCurrentState();
}

// Third part
if ((proposal.total_vote_to_end - total_vote == 1) && (choice == 1 || choice == 2 || choice == 0)) {
    proposal.is_active = false;

}

Let's break down the three parts of our function logic.


- In the first part, we are creating a storage variable which will be pointing to the current proposal from the proposal_history mapping. Any change we make on the storage variable proposal will be reflect to the current proposal. Then we are calculating the total_vote so that at the end we can check if the total vote to end this tutorial has finished with this final vote or not.


- In the second part, based on the choice from the user, we are incrementing the corresponding field of our proposal struct. For an example if the choice is 1, which means the caller wants to vote for approve, we say proposal.approve += 1 which increases the number of approve votes by one. After incrementing the field we calculate the state of the proposal with the line proposal.current_state = calculateState();. The method calculateState() may seem new but do not worry, we have not implemented this method yet, we will implement it after finishing the vote function.


- On the last part of the function we are checking if we needed just one vote to end the tutorial in the beginning of the function where we calculated the total votes. If this is the case and we got a parameter which is either 1, 2 or 0, we can end the proposal by setting its active field to false.


At this point, even though our syntax is correct, we have 2 big problems with our logic. Can you see them?

---

### Correcting The Voting Functionality

Last lesson, we said that we had 2 problems in our logic.


 The first problem is that a user can vote for a proposal even it is not active. 
 The second problem is that, a user can vote to the same proposal over and over again. 

We will be solving this issues with the help of the modifiers.


The first modifier we are going to be creating will check whether the current proposal is active or not.

```
modifier active() {
    require(proposal_history[_counter.current()].is_active == true, "The proposal is not active");
    _;
}
```

The modifier checks whether the proposal is active or not. If the proposal is not active, it returns the message "The proposal is not active".


This modifier will check if the voter has voted before. But to check this we need a keep the addresses that has voted for our contract. We can achieve this with an array.


Let's create the following variable under the mapping.


address[]  private voted_addresses;


Next, we need to modify our constructor so that we can add the owner of the contract to the array since the owner of the contract will be the one who creates the proposals and we do not want him/her to vote on the contracts that himself/herself has created.


We will be adding the following line at the end of our contructor function:


voted_addresses.push(msg.sender);


Now, our constructor becomes:

```
constructor() {
    owner = msg.sender;
    voted_addresses.push(msg.sender);
}
```

We also need to modify our vote function. At the third part (end) of the function we set active  field of the Proposal struct to false. Under that line we will also reset the that are voted:

```
if ((proposal.total_vote_to_end - total_vote == 1) && (choice == 1 || choice == 2 || choice == 0)) {
    proposal.is_active = false;
    voted_addresses = [owner];
}
```

Also before the if-else statements we will add the address to the array:


voted_addresses.push(msg.sender);


So, our vote function becomes:

```
function vote(uint8 choice) external active newVoter(msg.sender){
    Proposal storage proposal = proposal_history[_counter.current()];
    uint256 total_vote = proposal.approve + proposal.reject + proposal.pass;

    voted_addresses.push(msg.sender);

    if (choice == 1) {
        proposal.approve += 1;
        proposal.current_state = calculateCurrentState();
    } else if (choice == 2) {
        proposal.reject += 1;
        proposal.current_state = calculateCurrentState();
    } else if (choice == 0) {
        proposal.pass += 1;
        proposal.current_state = calculateCurrentState();
    }

    if ((proposal.total_vote_to_end - total_vote == 1) && (choice == 1 || choice == 2 || choice == 0)) {
        proposal.is_active = false;
        voted_addresses = [owner];
    }
}
```

Notice that, in the function definition, we also added the modifiers active and newVoter. We have not created the newVoter modifier, so let's create that one too.

```
modifier newVoter(address _address) {
    require(!isVoted(_address), "Address has not voted yet");
    _;
}
```

You may have noticed that we have used the function isVoted(). Do not worry if it looks unfamiliar, since we have not implement this function yet, but we will be in the future lessons. For now, just know that this function checks if the given address has voted or not.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Counters.sol";


contract ProposalContract {
    // ****************** Data ***********************

    //Owner
    address owner;

    using Counters for Counters.Counter;
    Counters.Counter private _counter;

    struct Proposal {
        string description; // Description of the proposal
        uint256 approve; // Number of approve votes
        uint256 reject; // Number of reject votes
        uint256 pass; // Number of pass votes
        uint256 total_vote_to_end; // When the total votes in the proposal reaches this limit, proposal ends
        bool current_state; // This shows the current state of the proposal, meaning whether if passes of fails
        bool is_active; // This shows if others can vote to our contract
    }

    mapping(uint256 => Proposal) proposal_history; // Recordings of previous proposals

    address[] private voted_addresses; 

    //constructor
    constructor() {
        owner = msg.sender;
        voted_addresses.push(msg.sender);
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier active() {
        require(proposal_history[_counter.current()].is_active == true);
        _;
    }

    modifier newVoter(address _address) {
        require(!isVoted(_address), "Address has not voted yet");
        _;
    }


//     // ****************** Execute Functions ***********************


    function setOwner(address new_owner) external onlyOwner {
        owner = new_owner;
    }

    function create(string calldata _description, uint256 _total_vote_to_end) external onlyOwner {
        _counter.increment();
        proposal_history[_counter.current()] = Proposal(_description, 0, 0, 0, _total_vote_to_end, false, true);
    }
    

    function vote(uint8 choice) external active newVoter(msg.sender){
        Proposal storage proposal = proposal_history[_counter.current()];
        uint256 total_vote = proposal.approve + proposal.reject + proposal.pass;

        voted_addresses.push(msg.sender);

        if (choice == 1) {
            proposal.approve += 1;
            proposal.current_state = calculateCurrentState();
        } else if (choice == 2) {
            proposal.reject += 1;
            proposal.current_state = calculateCurrentState();
        } else if (choice == 0) {
            proposal.pass += 1;
            proposal.current_state = calculateCurrentState();
        }

        if ((proposal.total_vote_to_end - total_vote == 1) && (choice == 1 || choice == 2 || choice == 0)) {
            proposal.is_active = false;
            voted_addresses = [owner];
        }
    }
}
```

In the next section we will be implementing the calculateCurrentState function to help us to retrieve the state of the current proposal.

---

### Calculating The Current State Of A Proposal

Before moving forward with the function implementation, let's check our the logic of calculating the current state of the contract.


We have three different choices:


- Approve


- Reject


- Pass


For a proposal to succeed, the number of approve votes should be higher than the sum of reject and half the pass votes (you can think like pass vote has the half the impact). So, our formula is: approve = reject + pass / 2.


We may have odd number of pass votes which cannot be divided by two. In these cases, we add 1 to the number of pass votes and then divide.


After knowing the logic of the function, the implementation is pretty straight forward:

```
function calculateCurrentState() private view returns(bool) {
    Proposal storage proposal = proposal_history[_counter.current()];

    uint256 approve = proposal.approve;
    uint256 reject = proposal.reject;
    uint256 pass = proposal.pass;
        
    if (proposal.pass %2 == 1) {
        pass += 1;
    }

    pass = pass / 2;

    if (approve > reject + pass) {
        return true;
    } else {
        return false;
    }
}
```

If you check the function definition, we see that we have used private and view.


- We used private because this function is just a helper function for our previous vote function and it is only being used in the contract.


- We used view because the function only views data from the blockchain and does not alter it.

---

### Calculating The Current State Of A Proposal

Before moving forward with the function implementation, let's check our the logic of calculating the current state of the contract.


We have three different choices:


- Approve


- Reject


- Pass


For a proposal to succeed, the number of approve votes should be higher than the sum of reject and half the pass votes (you can think like pass vote has the half the impact). So, our formula is: approve = reject + pass / 2.


We may have odd number of pass votes which cannot be divided by two. In these cases, we add 1 to the number of pass votes and then divide.


After knowing the logic of the function, the implementation is pretty straight forward:

```
function calculateCurrentState() private view returns(bool) {
    Proposal storage proposal = proposal_history[_counter.current()];

    uint256 approve = proposal.approve;
    uint256 reject = proposal.reject;
    uint256 pass = proposal.pass;
        
    if (proposal.pass %2 == 1) {
        pass += 1;
    }

    pass = pass / 2;

    if (approve > reject + pass) {
        return true;
    } else {
        return false;
    }
}
```

If you check the function definition, we see that we have used private and view.


- We used private because this function is just a helper function for our previous vote function and it is only being used in the contract.


- We used view because the function only views data from the blockchain and does not alter it.
