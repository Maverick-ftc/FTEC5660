# FTEC5660 Homework 1: Receipt Chain

Build a LangChain pipeline that reads every supermarket receipt in a folder
with the vision-capable DeepSeek Flash model and answers these two questions:

1. How much money did I spend in total for these bills?
2. How much would I have had to pay without the discount?

For this homework, **amount spent** means the final payment after the receipt's
rounding line. **Without the discount** means the sum of the original positive
item prices: add back every promotion, coupon, member, app, packaging-damage,
and percentage discount, but do not add back rounding.

## Student task

Only edit the two functions in `hw1.py` that contain `### YOUR CODE HERE`:

- `build_chain()` creates your LangChain chain.
- `answer_queries()` runs the chain on the receipt images and returns one final
  response for each question.

You may use prompt chaining, routing, parallel calls, reflection, or a
combination. Your final responses should each contain one HKD amount. Do not
hard-code filenames or public answers; grading uses unseen receipt folders.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your DeepSeek key after `DEEPSEEK_API_KEY=` in `.env`, then run:

```bash
python3 hw1.py --image-folder public_test
```

The program creates `results.csv` in the current directory. Its columns are
`query`, `model_response`, and `correctness`. The public answers are in
`public_test/ground_truth.json`. The starter intentionally returns the dummy
response `please design your chain to answer these two queries.` so it runs
before you add any API code.

The required model is `deepseek-v4-flash-vision-exp`, the vision-capable
DeepSeek Flash model. JPEG, PNG, GIF, and WebP inputs are accepted by the
homework runner.


## Homework 1 solution: 
> to students:
+-------------------------------------------------------------+
|                     Input: Receipt Image                    |
+-------------------------------------------------------------+
|
v
+-------------------------------------------------------------+
|              Single-Pass Multimodal Vision Chain            |
|     (deepseek-v4-flash-vision-exp + Single Structured Prompt) |
|                                                             |
|  * Extract paid_amount (after discount & rounding)          |
|  * Extract original_amount (subtotal + discounts back)      |
+-------------------------------------------------------------+
|
v
+-------------------------------------------------------------+
|                   Output: Raw JSON String                   |
|         {"paid_amount": 102.30, "original_amount": 107.70}  |
+-------------------------------------------------------------+
|
v
+-------------------------------------------------------------+
|             Python Post-Processing & Aggregation            |
|                                                             |
|  * Parse JSON using regex & fallback string handling        |
|  * Exact Decimal summation across all receipts              |
|  * Format output: HK$XXX.XX                                 |
+-------------------------------------------------------------+
To solve the task, I initially explored a multi-prompt pipeline as introduced in class (e.g., separating transcription and mathematical calculation steps). However, experiments showed that splitting prompts led to context loss and cumulative inaccuracies in discount line items. Ultimately, I settled on a single-pass multimodal prompt design leveraging DeepSeek's vision model. By providing a precise definition for both `paid_amount` and `original_amount` along with a concrete few-shot example, the model directly processes visual layout and spatial information in one step. The output is strictly formatted as JSON, allowing Python to reliably parse and aggregate the exact monetary amounts using `Decimal` arithmetic.

