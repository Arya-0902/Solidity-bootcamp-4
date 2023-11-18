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
