# ScopeMatch

A standalone, browser-local AI intake tool for freelancers and small studios.
It matches a short English-language client brief against six service packages,
then calculates an explainable draft price range from an hourly rate and buffer.

Created during Hacktoberfest 2026 Week 2: **Side Hustle**.

## Try it

**[Open the live demo](https://rahulatrkm.github.io/scopematch-hacktoberfest-2026/)**

Or serve this folder with any static HTTP server.
For example, with Python already installed:

```powershell
python -m http.server 8000 --bind 127.0.0.1
```

Visit `http://127.0.0.1:8000`. No build, account, API key, or package installation
is required. Use a recent desktop Chromium browser or Firefox with WebAssembly
support. Mobile devices may have less memory available.

1. Enter a 20-800 character brief or choose **Try a sample brief**.
2. Set your rate, currency, and contingency buffer.
3. Review the three semantic matches and choose a service.
4. Review the exclusions and follow-up questions before exporting a text draft.

The sample bakery brief should shortlist the landing-page service. The app does
not claim that any match or estimated number of hours is correct for a client.

## Where the AI actually runs

The core is the open-weight `Xenova/all-MiniLM-L6-v2` sentence embedding model,
executed with Transformers.js 3.8.1 and ONNX Runtime Web in the browser.
The model revision is pinned to
`751bff37182d3f1213fa05d7196b954e230abad9`.

The brief and six service descriptions become normalized, 384-dimensional
embeddings. Cosine similarity ranks the services. The app displays the scores
as similarities, never as calibrated confidence percentages.

The first analysis downloads the public library, runtime, and quantized model
assets from jsDelivr and Hugging Face. This can take tens of MB. A cold start
therefore needs internet access. Browser asset caching is not an offline-use
guarantee.

**Client briefs are not sent to an inference API, stored in local storage, or
collected by analytics.** They remain in page memory until the tab closes or the
page reloads. A user-requested quote download includes the brief on that user's
device. Asset providers still receive ordinary network metadata for downloads.

## Honest quoting

Price is deterministic:

```text
round(starter hours * hourly rate in cents * (1 + buffer / 100))
```

The model never invents an hourly rate or a price. Catalog hour ranges are
illustrative assumptions, not AI predictions. Prices exclude taxes,
subscriptions, and third-party costs. Quotes require an explicit human
selection and are labeled drafts.

Updating the brief invalidates earlier results. Invalid prices disable exports.
Model failures produce visible errors; there are no fake or keyword-based
"AI" fallback results.

## Who might pay for it?

Freelancers who repeatedly sell similar packages could adapt the catalog to
their own offers and use the tool to shorten first-call preparation. A small
studio could pay for a branded catalog, onboarding, and integration into its
intake process. These are potential business models, not claims of actual
revenue, customer validation, or paying users.

## Limitations

- Six sample services cannot represent every kind of freelance work.
- MiniLM here is intended for English semantic similarity, not negotiation,
  legal compliance, detailed requirements extraction, or reliable negation
  handling.
- Similarity below 0.30 triggers a low-overlap warning. This is a transparent
  heuristic, not a calibrated business decision boundary.
- Long or unusually token-dense inputs may be truncated by the model tokenizer;
  use a focused summary rather than a full specification.
- A first use on a slow connection or low-memory device may fail explicitly.
- Open-source dependencies are fetched from public providers. Audit and
  self-host them if your deployment has stricter supply-chain requirements.
- The application is a prototype, not a billing or legal-contract system.

## Customize

Edit the `SERVICES` constant in [index.html](./index.html) to describe your real
services, realistic hour ranges, exclusions, and discovery questions. Matching
and quote calculations use the same catalog.

The HTML is a standalone exportable artifact, not a scaffold requiring a local
project environment.

## Browser validation

The following were exercised against the actual application, not a mocked
embedding model:

| Check | Observed result |
| --- | --- |
| Bakery website brief | Landing page ranked first, cosine similarity 0.526 |
| Accessibility brief with the browser offline after model load | Accessibility audit ranked first, cosine 0.945 |
| USD 75/hour, 8-16 hours, 15% buffer | USD 690.00-1,380.00 |
| USD 100/hour, 8-16 hours, 0% buffer | USD 800.00-1,600.00 |
| Invalid rates, fractional cents, buffer above 100, short/overlong briefs | Visible errors; no stale quote export |
| Unrelated puppy-training brief | Top score 0.204; low-overlap warning shown |
| Blocked library download | Visible failure, no simulated results; reload restored inference |
| Actual 390px viewport | One column; no horizontal overflow |
| External Microsoft Edge quote export | Completed 977-byte text file; draft, price, exclusions, and questions verified |

These examples demonstrate a prototype's behavior, not general model accuracy
or customer validation. The browser preview's download-event automation did not
observe a download, so file delivery was checked separately in external Edge.

## Attribution and licensing

- Original application code: MIT, copyright 2026 Rahul.
- [Transformers.js](https://github.com/huggingface/transformers.js): Apache-2.0.
- [ONNX Runtime](https://github.com/microsoft/onnxruntime): MIT.
- [all-MiniLM-L6-v2 ONNX weights](https://huggingface.co/Xenova/all-MiniLM-L6-v2):
  Apache-2.0, derived from
  [sentence-transformers/all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2).

Third-party models and libraries are downloaded at runtime, not copied into
this repository. Their own licenses continue to apply.

An AI assistant using Copilot SDK in VS Code assisted with implementation and
validation. No customer testimonials or claimed sales were generated.
