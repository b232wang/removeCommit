
1, Apply application

registry.apply:

```
/**
 * @notice Allows a user to start an application.
 * @notice Takes tokens from user and sets apply stage end time.
 * @param _project The project of a potential listing a user 
 * is applying to add to the registry
 * @param _amount The number of ERC20 tokens a user is willing to potentially stake
 */
function apply(string _project, uint _amount) external;
```

2, Challenge application 
```
/**
 * @notice Starts a poll for a project which is either
 * @notice in the apply stage or already in the whitelist.
 * @dev Tokens are taken from 
 * the challenger and the applicant's deposit is locked.
 * @param _project The project of an applicant's potential listing
 */
function challenge(string _project) external returns (uint challengeID) ;
```

vote:

```
/**    
 * @notice Loads _numTokens ERC20 tokens into the voting contract for one-to-one voting rights
 * @dev Assumes that msg.sender has approved voting contract to spend on their behalf
 * @param _numTokens The number of votingTokens desired in exchange for ERC20 tokens
 */
function requestVotingRights(uint _numTokens) external;
```

```
/**
  @notice Commits vote using hash of choice and secret salt to conceal vote until reveal
  @param _pollId Integer identifier associated with target poll
  @param _secretHash Commit keccak256 hash of voter's choice and salt (tightly packed in this order)
  @param _numTokens The number of tokens to be committed towards the target poll
  @param _prevPollId The ID of the poll that the user has voted the maximum number 
  of tokens in which is still less than or equal to numTokens 
 */
function commitVote(uint _pollId, bytes32 _secretHash, uint _numTokens, uint _prevPollId ) external;
```

reveal stage:
plcrVoting.revealStageActive
```
/**
 * @notice Checks if the reveal period is still active for the specified poll
 * @dev Checks isExpired for the specified poll's revealEndDate
 * @param _pollId Integer identifier associated with target poll
 */
function revealStageActive(uint _pollId) public view returns (bool active) ;
```

plcrVoting.revealVote
```
/**
 * @notice Reveals vote with choice and secret salt used in generating 
 * commitHash to attribute committed tokens
 * @param _pollId Integer identifier associated with target poll
 * @param _voteOption Vote choice used to generate commitHash for associated poll
 * @param _salt Secret number used to generate commitHash for associated poll
 */
function revealVote(uint _pollId, uint _voteOption, uint _salt) external ;
```

update status:

register.updateStatus
```
/**
 * @notice Updates a project's status from 'application' to 'listing'
 * or resolves a challenge if one exists.
 * @param _project The project whose status is being updated
 */
function updateStatus(string _project) public;
```

register.claimReward
```
/**
  @notice Called by a voter to claim his/her reward for each completed vote.
  @dev Someone must call updateStatus() before this can be called.
  @param _challengeID The pollId of the challenge a reward is being claimed for
  @param _salt The salt of a voter's commit hash in the given poll
 */
function claimReward(uint _challengeID, uint _salt) public ;
```

plcrVoting.withdrawVotingRights
```
/**
 * @notice Withdraw _numTokens ERC20 tokens from the voting contract, revoking these voting rights
 * @param _numTokens The number of ERC20 tokens desired in exchange for voting rights
 */
function withdrawVotingRights(uint _numTokens) external ;
```

3, add milestone
milestoneController.addMilestone
```
/**
 * Add a milestone
 * Can only be called if the first milestone is INACTIVE or there is no milestones
 * If it is the first milestone to be added, set state to "INACTIVE" and endTime to
 * now + length.
 *
 * @param namespace namespace of a project
 * @param length length of the milestone, >= 60 days
 * @param objs list of objectives' IPFS hash
 * @param objTypes list of objectives' type
 * @param objMaxRegulationRewards list of objectives' max regulation rewards
 */
function addMilestone(
    bytes32 namespace, 
    uint length, 
    bytes32[] objs, 
    bytes32[] objTypes, 
    uint[] objMaxRegulationRewards) external founderOnly(namespace)
```

4, Token sale 

Project owner should transfer their token to TokenCollector

tokenSale.startTokenSale
```
/**
 * Start a token sale after application has been accepted
 *
 * @param namespace namespace of the project
 * @param rate (uint) of token sale
 * @param token address of the project token
 */
function startTokenSale(bytes32 namespace, uint rate, address token) external founderOnly (namespace)
```

tokenSale.buyTokens
```
/**
 * Purchase tokens
 *
 * Withdraw msg.value * rate tokens from TokenCollector, then
 * transfer tokens to msg.sender
 *
 * @param namespace namespace of the project
 */
function buyTokens(bytes32 namespace) public payable {}
```

tokenSale.finalize
```
/**
 * Finalize a token sale, investors can not call buyTokens
 * after a sale has been finalized
 *
 * Calculate the token sale average price, and write it to storage
 *
 * @param namespace namespace of the project
 */
function finalize(bytes32 namespace) external founderOnly(namespace) ;
```

5, Activate

milestoneController.activate
```
/**
 * Activate a milestone
 * First check if the previous milestone can be set to state "COMPLETION" using
 *   require(finalize(namespace, milestoneId));
 * Then activate this milestone
 *
 * @param namespace namespace of a project
 * @param milestoneId milestoneId of a milestone of the project
 * @param weiLocked the amount of wei locked in the milestone
 * @param minStartTime the minimum starting time (unix timestamp)
 *     to start a poll
 * @param maxStartTime the maximum starting time (unix timestamp)
 *     to start a poll
 */
function activate(
        bytes32 namespace,
        uint milestoneId,
        uint weiLocked,
        uint minStartTime,
        uint maxStartTime) external founderOnly(namespace)
```


6, Reputation system rating

reputationSystem.startPoll
```
/**
 * Start a poll by any address.
 *
 * @param id the id of the reputation system
 *   id cannot be the global reputation system's id
 * @param pollId the id of the poll to start, generated using
 *     keccak256(abi.encodePacked(projectNameHash, milestoneNameHash))
 * @param delayLength the delayed block number for startBlock
 * @param pollLength Length of poll measured by block number
 */
function startPoll(bytes32 id, bytes32 pollId, uint delayLength, uint pollLength) public onlyNonGlobalReputationsSystemID(id) {}
```

reputationSystem.vote
```
/**
 * Delegates votes to a principal in a poll, called by proxies
 *
 * @param id the id of the reputation system
 *   id cannot be the global reputation system's id
 * @param member the address of a member (regulator)
 * @param contextType the context type of the reputation vector
 * @param pollId the id of the poll, from which we delegate votes
 * @param votesInWei the number of Wei to delegate
 */
function vote(
        bytes32 id,
        address member,
        bytes32 contextType,
        bytes32 pollId,
        uint votesInWei
        ) public onlyNonGlobalReputationsSystemID(id)
```


milestoneController.startRatingStage
```
/**
 * Start rating stage (RS)
 * RS can be started by project founders at any time between startTime and endTime - 30 days.
 * Set state to "RS",  call module
 *      RegulatingRatingModule.start(namespace, milestoneId, objs, now, endTime - 30 days);
 *
 *  @param namespace namespace of a project
 *  @param milestoneId milestoneId of a milestone of the project
 */
function startRatingStage(bytes32 namespace, uint milestoneId) external founderOnly(namespace) {}
```

regulatingRating.bid
```
/**
 * Bid for objective, which is called by regulator
 *
 * @param namespace namespace of a project
 * @param milestoneId milestoneId of a milestone of the project
 * @param obj an objective of a milestone of the project
 */
function bid(bytes32 namespace, uint milestoneId, bytes32 obj) external {}
```


regulatingRating.finalizeAllBids
```
/**
 * Finalize all the bids for objectives
 *
 * @param namespace namespace of a project
 * @param milestoneId milestoneId of a milestone of the project
 */
function finalizeAllBids(bytes32 namespace, uint milestoneId) external founderOnly(namespace) {}
```

regulatingRating.finalizeBidForObj
```
/**
 * Finalize the bid of an objective that in rating process
 *
 * @param namespace namespace of a project
 * @param milestoneId milestoneId of a milestone of the project
 * @param obj an objective of a milestone of the project
 */
function finalizeBidForObj(bytes32 namespace, uint milestoneId, bytes32 obj) external founderOnly(namespace)
```

rewardManager.withdraw
```
/*
 * Withdraw reward by regulator after Rating Stage of the milestone
 *
 * @param namespace namespace of the project
 * @param milestoneId id of a milestone
 * @param obj an objective of a milestone of the project
 */
function withdraw(bytes32 namespace, uint milestoneId, bytes32 obj) external
```


7, Refund stage
milestoneController.startRefundStage
```
/**
 * Start a refund
 * Can be called at anytime between [endTime - 1 week, endTime) by any address
 * Set state to "RP"
 *
 * @param namespace namespace of a project
 * @param milestoneId milestoneId of a milestone of the project
 */
function startRefundStage(bytes32 namespace, uint milestoneId) external  ;
```

refundManager.refund
```
/*
 * Refund investors during milestone's refund stage
 * Assume msg.sender has approved "val" tokens for this contract
 * First transfer "val" tokens from msg.sender to TokenCollector
 * Then write down the the number of weis to be refunded, using
 * the ICO average price from TokenSale.
 * The refund will be locked for 1 month, after the freeze period,
 * investors can then withdraw their refunds *
 * @param namespace namespace of the project
 * @param milestoneId id of a milestone
 * @val amount of tokens to be returned
 */
function refund(bytes32 namespace, uint milestoneId, uint val) external {}
```

refundManager.withdraw
```
/* * Withdraw refunds after freeze period
 *
 * @param namespace namespace of the project
 * @param milestoneId id of a milestone
 */
function withdraw(bytes32 namespace, uint milestoneId) external {}
```




