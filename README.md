# Black-Scholes Options Pricing & Implied Volatility Engine

A Python implementation of the Black-Scholes model for pricing European options, with an Implied Volatility solver built on **Brent's method** and a Greeks calculator for risk management.

Built and tested against live NSE options data.

---

## What This Does

| Module | Description |
|--------|-------------|
| `black_scholes_call / put` | Theoretical price for European call and put options |
| `implied_volatility_brent` | Backs out IV from market price using Brent's root-finding method |
| `greeks` | Computes Delta, Gamma, Vega, Theta |

---

## Why Brent's Method — Not Newton-Raphson

This is a deliberate design choice, not an accident.

Newton-Raphson updates sigma using:

$$\sigma_{n+1} = \sigma_n - \frac{BS(\sigma_n) - C_{market}}{Vega(\sigma_n)}$$

**The problem:** For deep ITM or deep OTM options, Vega → 0. The denominator blows up, producing numerically unstable or non-convergent results.

**Brent's method** is derivative-free. It brackets the root between `[sigma_low, sigma_high]` and uses a combination of bisection, secant, and inverse quadratic interpolation to converge. It is guaranteed to find the root as long as:

- The bracket is valid: `f(sigma_low)` and `f(sigma_high)` have opposite signs
- Volatility is bounded: `sigma ∈ (0, 5.0]` — lower bound enforced because volatility is a standard deviation (must be > 0); upper bound of 500% is a practical ceiling no liquid instrument has breached

---

## Black-Scholes Formula

$$C = S \cdot N(d_1) - K e^{-rT} \cdot N(d_2)$$

$$P = K e^{-rT} \cdot N(-d_2) - S \cdot N(-d_1)$$

Where:

$$d_1 = \frac{\ln(S/K) + (r + \frac{\sigma^2}{2})T}{\sigma\sqrt{T}}, \quad d_2 = d_1 - \sigma\sqrt{T}$$

---

## Greeks

| Greek | Meaning | Formula |
|-------|---------|---------|
| **Delta** | Sensitivity of option price to spot price | $N(d_1)$ for call, $N(d_1)-1$ for put |
| **Gamma** | Rate of change of Delta | $\frac{N'(d_1)}{S\sigma\sqrt{T}}$ |
| **Vega** | Sensitivity to volatility (per 1% move) | $S \cdot N'(d_1) \cdot \sqrt{T} / 100$ |
| **Theta** | Time decay (per calendar day) | See notebook |

---

## Model Assumptions & Limitations

Black-Scholes makes assumptions that do not hold in real markets. These are documented honestly:

- **Constant volatility** — IV is not constant; real markets exhibit volatility smile/skew
- **European options only** — does not handle early exercise (American options)
- **No dividends** — model requires adjustment for dividend-paying stocks
- **Log-normal returns** — fat tails and jumps are not captured
- **Continuous trading** — real markets have gaps, liquidity constraints

This engine is accurate for pricing near-ATM European options. Edge cases (deep ITM/OTM, very short expiry) should be treated with caution.

---

## Sample Output

```
── ICICI Bank Call ──
  Market Price     : ₹20.40
  BS Price (IV)    : ₹20.40
  Implied Vol      : 19.3%
  Greeks           : {'Delta': 0.6231, 'Gamma': 0.0142, 'Vega': 0.0381, 'Theta': -0.8214}

── Put-Call Parity Verification ──
  Parity Holds     : True
```

---

## Tech Stack

- Python 3.x
- `numpy` — numerical computation
- `scipy.stats.norm` — normal distribution CDF/PDF
- `scipy.optimize.brentq` — Brent's root-finding method

---

## Related Projects

- [Pairs Trading — Walk-Forward Statistical Arbitrage Engine](https://github.com/devil0026/pairs-trading-nse)
- Commodity Derivatives Pricing Engine (Black '76 model) — coming soon

---

## Author

**Rakshit Gandhi**  
B.Sc. Information Technology | GLSU Ahmedabad  
Focused on quantitative finance, derivatives pricing, and systematic strategy development.  
[rakshitgandhi184@gmail.com](mailto:rakshitgandhi184@gmail.com)
