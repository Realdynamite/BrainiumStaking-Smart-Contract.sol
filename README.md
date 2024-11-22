// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StakingContract {
    struct Stake {
        uint256 amount;
        uint256 timestamp;
    }

    mapping(address => Stake) private stakes;

    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount, bool earlyUnstake, uint256 rewardOrPenalty);

    function stake() external payable {
        require(msg.value > 0, "Staking amount must be greater than 0");
        require(stakes[msg.sender].amount == 0, "You already have an active stake");

        stakes[msg.sender] = Stake(msg.value, block.timestamp);
        emit Staked(msg.sender, msg.value);
    }

    function unstake() external {
        Stake storage userStake = stakes[msg.sender];
        require(userStake.amount > 0, "No active stake found");

        uint256 stakedAmount = userStake.amount;
        uint256 stakingTime = block.timestamp - userStake.timestamp;

        uint256 payout;
        if (stakingTime >= 3 minutes) {
            // Reward the user with a 10% profit
            uint256 profit = (stakedAmount * 10) / 100;
            payout = stakedAmount + profit;
            emit Unstaked(msg.sender, stakedAmount, false, profit);
        } else {
            // Penalize the user with a 3% deduction
            uint256 penalty = (stakedAmount * 3) / 100;
            payout = stakedAmount - penalty;
            emit Unstaked(msg.sender, stakedAmount, true, penalty);
        }

        // Reset the stake
        delete stakes[msg.sender];

        // Send the payout to the user
        (bool success, ) = msg.sender.call{value: payout}("");
        require(success, "Transfer failed");
    }

    function getStake() external view returns (uint256 amount, uint256 timestamp) {
        Stake memory userStake = stakes[msg.sender];
        return (userStake.amount, userStake.timestamp);
    }

    function getContractBalance() external view returns (uint256) {
        return address(this).balance;
    }

    function getPenalty() public view returns (uint256) {
        Stake memory userStake = stakes[msg.sender];
        require(userStake.amount > 0, "No active stake found");

        uint256 penalty = (userStake.amount * 3) / 100;
        return penalty;
    }
}
