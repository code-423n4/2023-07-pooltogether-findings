**Steps to reproduce**

1. Install `vertigo-rs`.
2. Run `cd pt-v5-claimer`.
3. Run `python <path-to-this-project>/vertigo-rs/vertigo.py run`.
   The list of mutations will be generated.

To reproduce, replace the original line with the mutated line and run the tests.

For example, replace the `>` comparator with `>=` and run the tests: tests will pass.

Here is the full report:

<code>
Mutation testing report:
Number of mutations:    52
Killed:                 38 / 52

Mutations:

[+] Survivors
Mutation:
    File: /projects/pt-v5-claimer/src/Claimer.sol
    Line nr: 209
    Result: Lived
    Original line:
             return fee > _maxFee ? _maxFee : fee;

    Mutated line:
             return fee >= _maxFee ? _maxFee : fee;

Mutation:
    File: /projects/pt-v5-claimer/src/libraries/LinearVRGDALib.sol
    Line nr: 55
    Result: Lived
    Original line:
                 targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped

    Mutated line:
                 targetTimeUnwrapped > 0 && decayConstantUnwrapped >= type(int256).max / targetTimeUnwrapped

Mutation:
    File: /projects/pt-v5-claimer/src/libraries/LinearVRGDALib.sol
    Line nr: 61
    Result: Lived
    Original line:
                 targetTimeUnwrapped < 0 && decayConstantUnwrapped > type(int256).min / targetTimeUnwrapped

    Mutated line:
                 targetTimeUnwrapped < 0 && decayConstantUnwrapped >= type(int256).min / targetTimeUnwrapped

Mutation:
    File: /projects/pt-v5-claimer/src/libraries/LinearVRGDALib.sol
    Line nr: 79
    Result: Lived
    Original line:
               if (expResult > 1e18) {

    Mutated line:
               if (expResult >= 1e18) {

Mutation:
    File: /projects/pt-v5-claimer/src/libraries/LinearVRGDALib.sol
    Line nr: 88
    Result: Lived
    Original line:
                 if (targetPriceInt > type(int256).max / extraPrecisionExpResult) {

    Mutated line:
                 if (targetPriceInt >= type(int256).max / extraPrecisionExpResult) {

Mutation:
    File: /projects/pt-v5-claimer/src/libraries/LinearVRGDALib.sol
    Line nr: 81
    Result: Lived
    Original line:
                 if (targetPriceInt > type(int256).max / expResult) {

    Mutated line:
                 if (targetPriceInt >= type(int256).max / expResult) {

[*] Done! 

</code>
