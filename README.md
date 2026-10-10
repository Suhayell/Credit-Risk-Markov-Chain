# Credit-Risk-Markov-Chain

*Modelling loan default risk with an absorbing Markov chain in Python.*

## What we're building

The central idea is to compare two things:

- **Simulation:** We start with a transition matrix whose probabilities we already know, then generate thousands of synthetic loan histories.
- **Estimation:** We use the simulated histories to estimate the transition probabilities, then compare our estimates with the probabilities we started with.

If the estimates are close to the true values, the method is validated, and we can see how accuracy improves as the number of loans grows. We then use the estimated matrix to compute default probabilities over time and the expected time until a loan defaults or is repaid.

> All data and probabilities in this project are hypothetical. They are not based on real bank records.

## The model

Each loan is observed once a month and is in one of five states:

| Symbol | State | Type |
|---|---|---|
| C | Current (paying on time) | transient |
| L | Late (missed a payment) | transient |
| V | Very late (missed several payments) | transient |
| D | Default | absorbing |
| F | Paid off | absorbing |

The **Markov assumption** is that the next state depends only on the current state. The model is therefore described by a transition matrix $P$, where $P_{ij}$ is the probability of moving from state $i$ to state $j$ in one month. Default and Paid off are absorbing: once a loan enters either one, it stays there.

## Method

1. **Simulate.** Generate 10,000 loans over 36 months from a known matrix $P$ (vectorised with NumPy).
2. **Estimate.** Recover the matrix by counting transitions: $\hat P_{ij} = N_{ij} / \sum_k N_{ik}$, the maximum likelihood estimator.
3. **Validate.** Compare $\hat P$ with $P$, and measure how the estimation error falls as the number of loans grows.
4. **Analyse.** Use absorbing-chain theory on both $P$ and $\hat P$:
   - Cumulative default probability by month $n$ from the matrix power $P^n$.
   - Expected months until default or repayment from the fundamental matrix $N = (I - Q)^{-1}$.
   - Probability of eventually ending in default or repayment from $B = NR$.

## Key results

Estimation error (largest absolute error in $\hat P$, averaged over 20 repeats):

| Number of loans | Error |
|---|---|
| 100 | 0.063 |
| 1,000 | 0.018 |
| 10,000 | 0.006 |
| 100,000 | 0.002 |

The error falls roughly like $1/\sqrt{n}$, as theory predicts.

Results for the hypothetical matrix (estimated values agree with the true ones to within about 0.5 percentage points):

| Starting state | Expected months until default or repayment | Probability of eventual default |
|---|---|---|
| Current | 14.3 | 53.8% |
| Late | 10.2 | 57.7% |
| Very late | 6.3 | 65.4% |

A loan that starts Current has about a 50.5% chance of having defaulted by month 36.

## Repository structure

```
Credit-Risk-Markov-Chain/
├── notebooks/
│   └── credit_risk_analysis.ipynb   # full analysis, start to finish
├── requirements.txt
└── README.md
```

## How to run

```bash
git clone https://github.com/Suhayell/Credit-Risk-Markov-Chain.git
cd Credit-Risk-Markov-Chain
pip install -r requirements.txt
jupyter notebook notebooks/credit_risk_analysis.ipynb
```

A fixed random seed (`SEED = 42`) makes the results reproducible.

## Limitations

- **Hypothetical data.** The matrix is invented. Real probabilities are estimated from historical loan data and are not known in advance.
- **Markov assumption.** A loan with a history of repeated lateness probably behaves differently from one that was late once, but a first-order chain ignores that history.
- **Constant probabilities.** The same matrix is used every month, whereas real default risk depends on loan age and the economy.
- **Identical loans.** Loan size, interest rate and borrower type do not affect the transitions.
- **No monetary loss.** The model gives default probabilities, not amounts lost.

## Possible extensions

- Add expected loss: probability of default × loan amount × loss fraction.
- Estimate the matrix from a real public loan dataset or a published credit rating migration matrix.
- Test the Markov assumption, for example by comparing two-step transition counts with $\hat P^2$.
- Use a time-dependent matrix that changes with loan age.

## Tools

Python, NumPy, pandas, Matplotlib, Jupyter.