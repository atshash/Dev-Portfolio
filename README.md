# Shash | Software Developer

**Mobile apps · JavaScript / TypeScript · React Native · Python · Algorithms**

I'm a computer science professional with experience building mobile applications, writing numerical algorithms, and developing interactive software. I enjoy the full process of development: translating requirements into interfaces and application logic, connecting data to features, and testing the behavior that matters.

This portfolio includes **Metabolic Pal**, a completed private mobile-app build, plus **three runnable technical demos** I created to showcase options modeling, systematic strategy backtesting, and browser-game engineering. The demos are portfolio examples—not claims about previously shipped commercial products.

## Project index


| Project | Technology | What it demonstrates |
| --- | --- | --- |
| **01 / Metabolic Pal** | React Native, TypeScript, Expo, SQLite | Mobile app architecture, data persistence, domain logic, testing |
| **02 / OptionLab** | JavaScript, Canvas, CSS | Mathematical modeling, financial calculations, interactive data visualization |
| **03 / CorrectionLab** | Python, HTML/SVG | Event-driven algorithms, execution timing, risk logic, backtesting |
| **04 / Night Shift** | JavaScript, Canvas, CSS | Game loops, collision detection, state management, responsive input |

---

# 01 / Metabolic Pal

**Completed private mobile-app build · React Native · TypeScript · Expo · SQLite**

Metabolic Pal is an offline-first fitness and energy-tracking mobile application for iOS and Android. Its functionality includes food and activity logging, weight trends, a daily dashboard, estimated energy expenditure, and locally stored data. The application is private, rather than publicly distributed.

I structured the code around separate UI, domain, and persistence layers. Here are selected real excerpts from the codebase.

### Example A · TypeScript exercise-calculation engine

A numerical estimator should be understandable and testable without running an app UI. The exercise module validates values, implements VO₂-based activity calculations, and distinguishes gross calories from energy above rest.

```ts
const valid = (value: number, min: number, max: number) => {
  if (!Number.isFinite(value) || value < min || value > max)
    throw new Error('Check the workout settings.');
};

export const mphToMetersPerMinute = (mph: number) => mph * 26.8224;
export const walkingVO2 = (speed: number, grade: number) =>
  0.1 * speed + 1.8 * speed * grade + 3.5;

export function oxygenCalories(vo2: number, kg: number, minutes: number) {
  valid(vo2, 3.5, 200);
  valid(kg, 25, 350);
  valid(minutes, 0.1, 1440);
  return {
    grossCalories: ((vo2 * kg) / 1000) * 5 * minutes,
    netAdditionalCalories: (((vo2 - 3.5) * kg) / 1000) * 5 * minutes,
  };
}
```

**Engineering value:** reusable TypeScript functions, scientific-unit conversions, clear boundaries, and defensive input validation.

### Example B · Connecting React Native UI to application state

The dashboard gets its information from shared app state and domain models rather than duplicating calculations in components.

```tsx
export function Today() {
  const { state, now, model } = useData();
  const budget = model!;
  const e = budget.energy;
  const b = budget.balances;
  const remaining = budget.remaining;

  return (
    <Screen title="Today" eyebrow={now.toLocaleDateString('en-US')}>
      <Card filled style={{ padding: 24, borderRadius: 26 }}>
        <Copy size={11} bold muted>CALORIES REMAINING</Copy>
        <Copy testID="remaining" size={54} bold>
          ~{fmt(rounded(remaining))}
        </Copy>
        <Progress value={e.intake / (e.total + b.spendable + b.eventBudget)} />
      </Card>
    </Screen>
  );
}
```

*Condensed from the app's `Dashboard.tsx` screen; supporting components and imports are omitted for readability.*

**Engineering value:** composable native screens, shared state, data-driven presentation, and UI test identifiers.

### Example C · Testing and persistence

The application uses SQLite repositories, migration handling, validated state updates, and automated unit/integration tests. A test of the activity estimator checks its numeric output against a known scenario:

```ts
test('specified 171.4 lb, 3 mph, 12%, 30 min treadmill regression', () => {
  const e = treadmill([segment], toKg(171.4, 'lb'));
  close(e.grossCalories, 337.36, 1);
  close(e.netAdditionalCalories, 296.54, 1);
});
```

**Engineering value:** verifying calculations with repeatable tests, maintaining persistent application state, and separating user-interface behavior from core application logic.

---

# 02 / OptionLab — Options Calculator

**Runnable browser demo · JavaScript · Quantitative finance · HTML Canvas**

OptionLab is an interactive pricing and payoff calculator built from scratch with vanilla JavaScript. It uses the **Black–Scholes–Merton model** for European-style options, taking in spot, strike, days to expiry, volatility, risk-free rate, and dividend yield.

**Features:** theoretical call/put premiums, Delta/Gamma/Theta/Vega/Rho, a numerical implied-volatility solver, no-arbitrage bounds, multi-contract costs, and an interactive expiration payoff graph.

### Code highlight · Pricing and option sensitivities

```js
const d1 = (Math.log(S / K) + (r - q + sigma*sigma/2)*T)
  / (sigma * Math.sqrt(T));
const d2 = d1 - sigma * Math.sqrt(T);

const callPrice = S * Math.exp(-q*T) * normalCDF(d1)
                - K * Math.exp(-r*T) * normalCDF(d2);

const delta = Math.exp(-q*T) * normalCDF(d1);
const gamma = Math.exp(-q*T) * normalPDF(d1)
              / (S * sigma * Math.sqrt(T));
```

**Why it's relevant:** the project demonstrates mathematical logic, a clean separation between model code and UI, and data visualization driven by live user input. It also has **9 automated tests**, including known pricing benchmarks and put–call parity.

---

# 03 / CorrectionLab — Correction-Rebound Trading Algorithm

**Runnable Python demo · Systematic signals · Risk controls · Backtesting**

CorrectionLab investigates a simple rule-based strategy: **identify a correction from a recent peak, wait for a possible rebound, and simulate a trade**. Unlike a script that looks only at returns, it models order timing, position size, stop conditions, transaction fees, slippage, and daily marked-to-market equity.

The example uses explicitly **fictional synthetic price data** and makes no claim about actual market profitability.

### The strategy

1. At the close, identify a price at least **10% below** the recent 20-session closing high.
2. Require a positive close-to-close move as a simple reversal filter.
3. Submit a long-entry signal, filled at the **next session's open** rather than at the already-observed close.
4. Evaluate a close-based risk stop, recovery target, or maximum holding period, with exits likewise filled at the **next open**.
5. Track realized closed-trade P/L, marked-to-market account value, maximum drawdown, and trade details.

### Code highlight · No-look-ahead correction signal

```python
def correction_rebound_signal(bars, index, cfg):
    if index < cfg.lookback - 1:
        return False, 0.0

    peak = max(
        b.close for b in bars[index - cfg.lookback + 1:index + 1]
    )
    qualifies = (
        bars[index].close <= peak * (1 - cfg.correction)
        and bars[index].close > bars[index - 1].close
    )
    return qualifies, peak
```

**Why it's relevant:** it demonstrates event-driven program design, handling historical data correctly, realistic execution assumptions, and separating an algorithm's research logic from evaluation. The project includes **8 automated tests** and generates a standalone interactive-looking HTML/SVG report.

---

# 04 / Night Shift — Arcade Racing Game

**JavaScript · Canvas 2D · Game physics**

Night Shift is a self-contained endless driving game where the player moves between three lanes to avoid traffic. It includes increasing speed/difficulty, collision detection, scorekeeping, best-score persistence, pause/restart controls, and keyboard and touch input.

### Code highlight · Pure, testable collision logic

```js
export function collision(a, b) {
  return Math.abs(a.x - b.x) < VEHICLE_WIDTH * .79
      && Math.abs(a.y - b.y) < VEHICLE_HEIGHT * .79;
}

export function changeLane(state, step) {
  if (!['playing', 'ready'].includes(state.status)) return state;
  return {
    ...state,
    lane: Math.max(0, Math.min(LANES.length - 1,
      state.lane + Math.sign(step))),
  };
}
```

The graphics are drawn with Canvas instead of using a game engine. The **game rules are kept separate from browser rendering**, so the underlying movement, collisions, obstacle spawning, and scoring can be tested without a DOM.

**Why it's relevant:** the project showcases JavaScript fluency, time-based game loops, user input, visual feedback, and implementation of a complete interactive experience. The game includes **8 automated tests**.

---

## My development approach

- **Understand the product.** Start with the user problem and define the expected behavior before implementing.
- **Build with clear boundaries.** Keep UI, data access, and reusable algorithms separately organized.
- **Test critical behavior.** Use automated tests for numerical calculations, execution timing, and state transitions.
- **Deliver iteratively.** Demonstrate small, reviewable features rather than assuming requirements won't change.

## Areas of interest

I'm especially interested in mobile application development, financial software, data-driven products, and interactive experiences. I enjoy working across the full feature lifecycle—from requirements and interface design to domain logic, persistence, validation, and tests.

**Core skills demonstrated here:** JavaScript, TypeScript, React Native, Expo, Python, SQLite, mathematical modeling, automated testing, HTML/CSS, and Canvas APIs.

---
