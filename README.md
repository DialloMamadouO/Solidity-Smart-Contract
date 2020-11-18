# Solidity-Smart-Contract
In this project, we will use an ethereum-compatible blockchain to build smart contracts and connect financial institutions to automate some company finances, increase transparency, and make accounting and auditing practically automatic. 
These smart contracts will enable the company to:

-Pay Associate-level employees quickly and easily.

-Distribute profits to different tiers of employees.

-Distribute company shares for employees in a "deferred equity incentive plan" automatically.

We will build three seperate smart contracts that we will designate as level one, level two and level three.

-Level One is an AssociateProfitSplitter contract. This will accept Ether into the contract and divide the Ether evenly among the associate level employees. This will allow the Human Resources department to pay employees quickly and efficiently.

-Level Two is a TieredProfitSplitter that will distribute different percentages of incoming Ether to employees at different tiers/levels. For example, the CEO gets paid 60%, CTO 25%, and Bob gets 15%.

-Level Three is a DeferredEquityPlan that models traditional company stock plans. This contract will automatically manage 1000 shares with an annual distribution of 250 over 4 years for a single employee.

## Starting the project
We will navigate to the Remix IDE and create a new contract called AssociateProfitSplitter.sol.

While developing and testing our contract, we will use the Ganache development chain, and point MetaMask to localhost:8545. 

### Level One: The AssociateProfitSplitter Contract

At the top of our contract, we will define the following public variables:


employee_one -- The address of the first employee will be set to payable.


employee_two -- Another address payable that represents the second employee.


employee_three -- The third address payable that represents the third employee.

contract AssociateProfitSplitter {
    address payable Owner;
    address payable employee_one;
    address payable employee_two;
    address payable employee_three;

Then, we create a constructor function that accepts:


address payable _one


address payable _two


address payable _three

Within the constructor, we will set the employee addresses to equal the parameter values. This will allow us to avoid hardcoding the employee addresses.

 constructor(address payable _one, address payable _two, address payable _three) public {
        Owner = msg.sender;
        employee_one = _one;
        employee_two = _two;
        employee_three = _three;
   
Next, we will create the following functions:

balance -- This function should be set to public view returns(uint), and must return the contract's current balance. 
Since we should always be sending Ether to the beneficiaries, this function should always return 0. 

 function balance() public view returns(uint) {
        return address(this).balance;
        
deposit -- This function should set to public payable and ensure that only the owner can call the function.

In this function, we will perform the following steps:

-Set a uint amount to equal msg.value / 3; in order to calculate the split value of the Ether.
-Transfer the amount to employee_one.
-Repeat the steps for employee_two and employee_three.

Since uint only contains positive whole numbers, and Solidity does not fully support float/decimals, we must deal with a potential remainder at the end of this function since amount will discard the remainder during division.
We may either have 1 or 2 wei leftover, so we will transfer the (msg.value - amount * 3) back to msg.sender. This will re-multiply the amount by 3, then subtract it from the msg.value to account for any leftover wei, and send it back to Human Resources.

 function deposit() public payable {
        uint amount = msg.value / 3;

        // transfer the amount to each employee
        employee_one.transfer(amount);
        employee_two.transfer(amount);
        employee_three.transfer(amount);
        
        // Take care of a potential remainderr by sending back to HR (msg.sender)
        msg.sender.transfer(msg.value - (amount * 3));


Finally, we will create a fallback function using function() external payable, and call the deposit function from within it. This will ensure that the logic in deposit executes if Ether is sent directly to the contract. This is important to prevent Ether from being locked in the contract since we don't have a withdraw function in this use-case.

 function fallback() external payable {
        deposit();

### Test the Contract
In the Deploy tab in Remix, we will deploy the contract to our local Ganache chain by connecting to Injected Web3 and ensuring MetaMask is pointed to localhost:8545.
We will fill in the constructor parameters with our designated employee addresses.
Test the deposit function by sending various values and keep an eye on the employee balances as we send different amounts of Ether to the contract and make sure the logic is executing properly (See screenshots).

### Level Two: The TieredProfitSplitter Contract
In this contract, rather than splitting the profits between Associate-level employees, we will calculate rudimentary percentages for different tiers of employees (CEO, CTO, and Bob).
We will perform the following:

-Calculate the number of points/units by dividing msg.value by 100.

This will allow us to multiply the points with a number representing a percentage. For example, points * 60 will output a number that is ~60% of the msg.value.

The uint amount variable will be used to store the amount to send each employee temporarily. For each employee, we will set the amount to equal the number of points multiplied by the percentage (say, 60 for 60%).

After calculating the amount for the first employee, we will add the amount to the total to keep a running total of how much of the msg.value we are distributing so far.

Then, we will transfer the amount to employee_one and repeat the steps for each employee, setting the amount to equal the points multiplied by their given percentage.

For example:

Step 1: amount = points * 60;
For employee_one, distribute points * 60.
For employee_two, distribute points * 25.
For employee_three, distribute points * 15.

Step 2: total += amount;

Step 3: employee_one.transfer(amount);

We will send the remainder to the employee with the highest percentage by subtracting total from msg.value, and sending that to an employee.

function deposit() public payable {
        uint points = msg.value / 100; // Calculates rudimentary percentage by dividing msg.value into 100 units
        uint total;
        uint amount;
        
        amount = points * 60;
        total += amount;
        employee_one.transfer(amount);
        
        amount = points * 25;
        total += amount;
        employee_two.transfer(amount);
        
        amount = points * 15;
        
        employee_three.transfer(amount);
        
        total += amount;
        
        employee_one.transfer(msg.value - total); // ceo gets the remaining wei


Then, we eill deploy and test the contract functionality by depositing various Ether values (greater than 100 wei).

The balance function can be used as a test to see if the logic we have in the deposit function is valid. Since all of the Ether should be transferred to employees, this function should always return 0, since the contract should never store Ether itself.

### Level Three: The DeferredEquityPlan Contract
In this contract, we will be managing an employee's "deferred equity incentive plan" in which 1000 shares will be distributed over 4 years to the employee. We won't need to work with Ether in this contract, but we will be storing and setting amounts that represent the number of distributed shares the employee owns and enforcing the vetting periods automatically.

Human Resources will be set in the constructor as the msg.sender, since HR will be deploying the contract.
Below the employee initialization variables we will set the total shares and annual distribution:

-Create a uint called total_shares and set this to 1000.

-Create another uint called annual_distribution and set this to 250. This equates to a 4 year vesting period for the total_shares, as 250 will be distributed per year. Since it is expensive to calculate this in Solidity, we can simply set these values manually. We can tweak them as we see fit, as long as we can divide total_shares by annual_distribution evenly.

The uint start_time = now; line permanently stores the contract's start date. We'll use this to calculate the vested shares later. 
Below this variable, we will set the unlock_time to equal now plus 365 days. We will increment each distribution period.

The uint public distributed_shares will track how many vested shares the employee has claimed and was distributed. By default, this is 0.

In the distribute function we will add the following require statements:

-Require that unlock_time is less than or equal to now.

-Require that distributed_shares is less than the total_shares the employee was set for.

-Ensure to provide error messages in our require statements.

After the require statements, we will add 365 days to the unlock_time. This will calculate next year's unlock time before distributing this year's shares. We want to perform all of our calculations like this before distributing the shares.

Next, we will set the new value for distributed_shares by calculating how many years have passed since start_time multiplied by annual_distributions. For example:

The distributed_shares is equal to (now - start_time) divided by 365 days, multiplied by the annual distribution. If now - start_time is less than 365 days, the output will be 0 since the remainder will be discarded. If it is something like 400 days, the output will equal 1, meaning distributed_shares would equal 250.
We will make sure to include the parenthesis around now - start_time in our calculation to ensure that the order of operations is followed properly.

The final if statement provided checks that in case the employee does not cash out until 5+ years after the contract start, the contract does not reward more than the total_shares agreed upon in the contract.

    function distribute(uint amount) public {
        require(msg.sender == HR || msg.sender == employee, "You are not authorized to execute this contract.");
        require(active == true, "Contract not active.");

        // @TODO: Add "require" statements to enforce that:
        // 1: `unlock_time` is less than or equal to `now`
        // 2: `distributed_shares` is less than the `total_shares`
        require(unlock_time <= now, "You are not vested!");
        require(distributed_shares <= total_shares, "There are no shares left!");
        

        // @TODO: Add 365 days to the `unlock_time`
        unlock_time = now + 365 days;

        // @TODO: Calculate the shares distributed by using the function (now - start_time) / 365 days * the annual distribution
        // Make sure to include the parenthesis around (now - start_time) to get accurate results!
        distributed_shares = (now - start_time) / 365 days * 250;

        // double check in case the employee does not cash out until after 5+ years
        if (distributed_shares > 1000) {
            distributed_shares = 1000;


We will deploy and test our contract locally.

For this contract, we will test the timelock functionality by adding a new variable called uint fakenow = now; as the first line of the contract, then replace every other instance of now with fakenow.
Let's utilize the following fastforward function to manipulate fakenow during testing.

We will add this function to "fast forward" time by 100 days when the contract is deployed (requires setting up fakenow):

function fastforward() public {
    fakenow += 100 days;
}

Once we are satisfied with our contract's logic, we will revert the fakenow testing logic.

### Deploying the contracts to a live Testnet
Finally, we will use Kovan network, switch MetaMask to Kovan and deploy the contracts as we did on our localhost and keep a note of their deployed addresses (see screenshots). 













 
