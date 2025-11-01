# Technical Skills Test — Token Presale dApp

**Role:** Full-Stack Web3 Developer (Mid Level)
**Project:** Token Presale dApp (React + Solidity + Node.js)
**Instructions:** Answer all questions in writing. Code snippets are encouraged where relevant. There are no trick questions.

---

## Project Overview

Please read this section carefully before answering any questions. A good understanding of what this project is and what it is trying to achieve is part of the assessment.

**ELO (Effortless Order)** is a decentralized reward and ordering ecosystem. The token presale dApp is the first phase of the platform — it allows early participants to purchase ELO tokens at a fixed rate before the token is publicly listed on exchanges.

**The platform is built around four pillars:**

- **Presale** — Users send ETH to the presale contract during a fixed time window and receive ELO tokens in return. If the softcap is not reached by the end of the presale, all ETH is refunded. If the hardcap is reached, the presale ends early.
- **Staking** — After the presale, token holders will be able to stake ELO to earn rewards over time across multiple lock-period pools.
- **DEX / CEX Listing** — A portion of the supply is reserved for liquidity provision on decentralized and centralized exchanges.
- **Reward & Airdrop** — Token allocations are reserved to reward customers and participants within the Effortless Order CRM ecosystem.

**Token: ELO**
- Total Supply: 1,000,000,000 (1 Billion)
- Decimals: 18
- Network: Binance Smart Chain (BSC)

**Token Distribution:**

| Allocation | Percentage |
|---|---|
| Presale | 10% |
| DEX Liquidity | 10% |
| CEX Reserved | 10% |
| Staking Rewards | 25% |
| Team | 10% |
| Order Rewards | 20% |
| Customer Rewards | 5% |
| Airdrop | 5% |
| **Total** | **100%** |

**Presale Parameters (BSC Testnet):**
- Token Rate: 100,000 ELO per 1 BNB
- Minimum contribution: 0.1 BNB
- Maximum contribution: 50 BNB
- Softcap: 400 BNB
- Hardcap: 1,000 BNB

**Before answering the technical questions below, briefly answer the following:**

> **Q0.** In your own words, describe how the presale lifecycle works end-to-end — from a user's first deposit through to either claiming their tokens (success) or receiving a refund (failure). What role does each smart contract play?

---

## Section 1 — Smart Contracts (Solidity)

**Q1.** Review this presale status logic:

```solidity
function presaleStatus() public view returns (uint256) {
    if ((block.timestamp > presale_info.presale_end) && (status.raised_amount < presale_info.softcap)) {
        return 3; // Failure
    }
    if (status.raised_amount >= presale_info.hardcap) {
        return 2; // Success
    }
    if ((block.timestamp > presale_info.presale_end) && (status.raised_amount >= presale_info.softcap)) {
        return 2; // Success
    }
    if ((block.timestamp >= presale_info.presale_start) && (block.timestamp <= presale_info.presale_end)) {
        return 1; // Active
    }
    return 0; // Queued
}
```

What happens if the hardcap is reached before the presale end time? Is the status logic ordered correctly? What edge case is missing?

---

**Q2.** Review this deposit function:

```solidity
function userDeposit() public payable nonReentrant {
    require(presaleStatus() == 1, "Not Active");
    require(presale_info.raise_min <= msg.value, "Balance is insufficent");
    require(presale_info.raise_max >= msg.value, "Balance is too much");

    BuyerInfo storage buyer = buyers[msg.sender];

    uint256 amount_in = msg.value;
    uint256 allowance = presale_info.raise_max.sub(buyer.base);
    uint256 remaining = presale_info.hardcap - status.raised_amount;

    allowance = allowance > remaining ? remaining : allowance;
    if (amount_in > allowance) {
        amount_in = allowance;
    }

    uint256 tokensSold = amount_in.mul(presale_info.token_rate);

    require(tokensSold > 0, "ZERO TOKENS");
    require(status.raised_amount * presale_info.token_rate <= IERC20(presale_info.sale_token).balanceOf(address(this)), "Token remain error");

    if (buyer.base == 0) {
        status.num_buyers++;
    }
    buyers[msg.sender].base = buyers[msg.sender].base.add(amount_in);
    buyers[msg.sender].sale = buyers[msg.sender].sale.add(tokensSold);
    status.raised_amount = status.raised_amount.add(amount_in);
    status.sold_amount = status.sold_amount.add(tokensSold);

    if (amount_in < msg.value) {
        msg.sender.transfer(msg.value.sub(amount_in));
    }

    emit UserDepsitedSuccess(msg.sender, msg.value);
}
```

Identify at least **two issues** in this function and explain how you would fix each one.

---

**Q3.** Review this refund function:

```solidity
function userWithdrawBaseTokens() public nonReentrant {
    require(presaleStatus() == 3, "Not failed.");

    BuyerInfo storage buyer = buyers[msg.sender];
    uint256 remainingBaseBalance = address(this).balance;

    require(remainingBaseBalance >= buyer.base, "Nothing to withdraw.");

    status.base_withdraw = status.base_withdraw.add(buyer.base);

    address payable reciver = payable(msg.sender);
    reciver.transfer(buyer.base);

    if (msg.sender == owner) {
        ownerWithdrawTokens();
    }

    buyer.base = 0;
    buyer.sale = 0;

    emit UserWithdrawSuccess(buyer.base);
}
```

There are at least **three bugs** here. List each one and explain how you would fix it.

---

**Q4.** The token contract distributes all allocations inside the constructor:

```solidity
constructor() {
    _mint(_msgSender(), _totalSupply);
    mintingFinishedPermanent = true;

    transfer(presaleAddress, _presaleAmount);
    transfer(stakingAddress, _stakingAmount);
    transfer(cexReservedAddress, _cexReseveredAmount);
    transfer(dexAddress, _dexAmount);
    transfer(rewardAddress, _rewardAmount);
    transfer(customerAddress, _customerAmount);
    transfer(airdropAddress, _airdropAmount);
}
```

What is the risk of performing all token distribution inside the constructor? How would you redesign this?

---

**Q5.** The `ownerWithdrawTokens` function emits an event after transferring tokens:

```solidity
function ownerWithdrawTokens() private onlyOwner {
    require(presaleStatus() == 3, "Only failed status.");
    TransferHelper.safeTransfer(
        address(presale_info.sale_token),
        owner,
        IERC20(presale_info.sale_token).balanceOf(address(this))
    );
    emit UserWithdrawSuccess(IERC20(presale_info.sale_token).balanceOf(address(this)));
}
```

What value will this event always emit, and why? How do you fix it?

---

## Section 2 — Web3 / Wallet Integration

**Q6.** A developer wrote this code to connect to a user's wallet and read their token balance:

```javascript
const provider = new ethers.providers.Web3Provider(window.ethereum);
const signer = provider.getSigner();
const contract = new ethers.Contract(TOKEN_ADDRESS, ABI, signer);
const balance = await contract.balanceOf(await signer.getAddress());
```

This code fails. What is the cause and how do you fix it?

---

**Q7.** A user calls `userDeposit()` on the presale contract from the frontend. How would you correctly send ETH along with the contract call using ethers.js v6? Write the code.

---

**Q8.** After a successful presale, the frontend should check that `presaleStatus()` returns `2` before enabling the "Claim Tokens" button. How would you read this view function and conditionally enable the button in a React component? Write the logic.

---

**Q9.** The dApp needs to listen for the `UserDepsitedSuccess` event in real time and update the total raised amount in the UI whenever a new deposit is made. How would you implement this using ethers.js v6?

---

## Section 3 — Frontend (React)

**Q10.** Build a wallet connection button in React. When clicked it should connect MetaMask, store the wallet address in state, and display a shortened version (e.g. `0x1234...abcd`) once connected. Write the component.

---

**Q11.** A user submits a deposit transaction. The transaction is confirmed on-chain but the UI does not reflect the new balance until the page is refreshed. How would you handle post-transaction UI updates correctly in React?

---

**Q12.** The presale has a countdown timer to the start date. The start timestamp is stored in the smart contract as a Unix timestamp. How would you fetch that value from the contract and display a live countdown in React?

---

## Section 4 — Backend (Node.js / API)

**Q13.** Write an Express API endpoint that returns the current presale stats — amount raised, number of buyers, and presale status — by reading directly from the deployed smart contract using ethers.js.

---

**Q14.** The backend needs to record each participating wallet address and their deposit amount for off-chain tracking whenever a deposit event is emitted on-chain. Design the data model and write the logic to save a new entry when the event is detected.

---

**Q15.** A frontend request to your backend API fails in the browser with a CORS error. Explain what is causing this and how you would configure Express to handle it correctly for a production dApp.

---

## Submission

Please send your answers as a written document (PDF, Markdown, or Google Doc).

**Time suggested:** 30–60 minutes

---

*We are looking for clear technical reasoning, not just correct answers. How you think through a problem matters as much as the solution itself.*