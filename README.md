# Overflow Lab

An interactive teaching tool for understanding **arithmetic (integer) overflow**. It runs entirely in the browser from a single file, `overflow-lab.html`, with no installs, no build step, no network requests and no external libraries.

> **Safety note:** Overflow Lab is a calculator and visualizer. It simulates fixed-width arithmetic with JavaScript `BigInt` values. It never reads or writes real memory, and it contains no exploit, shellcode or attack code.

## Quick start

1. Download `overflow-lab.html`.
2. Double-click it, or drag it into any modern browser (Chrome, Firefox, Safari, Edge).
3. That's it. It works offline.

## Learning goals

After using the tool, a learner should be able to explain:

- Why a fixed number of bits can only represent a limited range of values.
- How unsigned and signed (two's complement) integers differ in range and meaning.
- What "wrapping around" means, and why the result is the true answer modulo 2^n.
- Why the same bit pattern can mean two different numbers depending on interpretation.
- How an overflow in a size calculation (`count × item size`) can lead to a buffer that is too small, and how a pre-check prevents it.

## Tab 1: Wrap-around arithmetic

| Control | What it does |
|---|---|
| **Bit width** | Choose 4, 8, 16 or 32 bits. Smaller widths make overflow easy to trigger and see. |
| **Interpretation** | Switch between **Unsigned** (0 to 2^n − 1) and **Signed** (−2^(n−1) to 2^(n−1) − 1). Switching keeps the bit patterns and re-reads them, which shows the signed/unsigned conversion effect. |
| **Operation** | `A + B`, `A − B` or `A × B`. |
| **A slider / A and B boxes** | Set the operands. Values are clamped to the legal range for the current width. |
| **Clickable bits of A** | Click any bit to flip it and watch the value (and result) change. In signed mode the sign bit is outlined. |
| **Try an overflow** | Loads a set of values that overflow for the current settings. |

What you see:

- **Binary rows** for A, B and the stored result, with decimal and hex shown beside each.
- **Result banner** stating whether overflow occurred. If it did, it shows the true answer, the stored answer, and how many full laps of the number range separate them.
- **Readout** with the true result, stored result, binary form, and what the same bits would read as under the *other* interpretation.
- **Number ring** showing every possible value arranged in a circle. The red seam marks where the maximum wraps to the minimum. For add and subtract, an arc walks from **A** by ±**B**; if it crosses the seam, the result **R** lands on the far side. The arc turns amber on overflow. Multiplication jumps rather than walks, so only A and R are marked.

### Things to try

- 8-bit unsigned: set A = 200, B = 100, operation `+`. True result 300, stored result 44.
- 8-bit signed: set A = 127, B = 1, operation `+`. The stored result is −128. Positive plus positive gives a negative number.
- 8-bit unsigned: set A = 0, B = 1, operation `−`. The result wraps to 255 (an "underflow").
- Flip the interpretation toggle with A = 255 unsigned and watch it become −1 signed with the same bits.
- 4-bit width with `×`: small numbers overflow quickly, so patterns are easy to spot.

## Tab 2: Overflow in memory sizing

Programs often compute an allocation size as `count × bytes_per_item`. If that multiplication wraps in the size variable, the program reserves far fewer bytes than it intends to fill.

| Control | What it does |
|---|---|
| **Item count** and **Bytes per item** | The two factors in the size calculation. |
| **Size variable** | 8-, 16- or 32-bit unsigned type holding the product. |
| **Safety check** | Off: the multiplication runs unchecked. On: the program first tests `count > MAX ÷ size` and rejects the request if the product would not fit. |
| **Presets** | A 16-bit example, a 32-bit example, and a no-overflow example. |

The display shows the intended byte count, the byte count actually reserved after wrapping, and two bars comparing "bytes reserved" with "bytes the loop expects to fill". When they differ, it reports how many bytes would fall outside the buffer. Turning the safety check on shows the request rejected before any multiplication, which is the standard fix.

Example: with a 16-bit size variable, 4097 items × 16 bytes = 65,552 intended, but 65,552 mod 65,536 = 16 bytes reserved.

## How it works (for readers of the code)

- **Single file.** HTML, CSS and JavaScript all live in `overflow-lab.html`.
- **BigInt arithmetic.** All values use `BigInt`, so 32-bit multiplication and the 32-bit size example are exact and never lose precision.
- **Key helper functions** (in the `<script>` block):
  - `wrap(x)`: reduces a true result to the stored value for the current width and signedness (`x mod 2^n`, then re-read as signed if needed).
  - `uBits(x)`: the unsigned bit pattern of a value.
  - `interp(u)`: reads a bit pattern as signed or unsigned.
  - `drawBits(...)`: renders the clickable bit cells.
  - `render()` and `render2()`: redraw tab 1 and tab 2 from the current state.
- **Overflow detection** is simply `trueResult !== wrap(trueResult)`.
- **Ring placement.** A value's position is `(value − min) / 2^n` of a full turn, so the seam is always at the top between the maximum and the minimum.
- **Theming.** Colors are CSS variables with automatic light and dark modes.

## Extending it

Ideas for classroom or student projects:

- Add carry-flag and overflow-flag displays, as on a CPU status register.
- Add saturating arithmetic as a comparison mode.
- Add a quiz that gives A, B and a width and asks for the stored result.
- Show signed-to-unsigned conversion bugs, such as a negative length compared against an unsigned limit.
- Add a step-by-step animation of the binary addition with carries.

## Limitations

- Only 4, 8, 16 and 32-bit widths, and only add, subtract and multiply.
- It models two's complement wrapping, which is how most hardware behaves. Some languages treat signed overflow as *undefined behavior* (for example C and C++), so real program behavior can differ from this model.
- Tab 2 simulates the arithmetic only. It does not demonstrate memory corruption.

## Browser support

Any modern browser with `BigInt` support (all current versions of Chrome, Firefox, Safari and Edge).
