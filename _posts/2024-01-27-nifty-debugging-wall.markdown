---
layout: post-custom
title:  "The Nifty-Gritty: Debugging-Wall"
date:   2024-01-26 11:06:58 +0100
categories: [science-stuff-nifty]
---
{% include katexLink.html%}

A collection of problems occurring to me in NIFTy and how I solved them.

- If you implement a new response operator, make sure that in the `mode=TIMES` the returned field is defined on data space (Unstructured Domain)
- For the `mode=ADJOINT_TIMES`, make sure the returned field is defined on the signal space (e.g. RGSpace)  
- Implement class properties `domain` and `target` that return `self._domain` and `self._target`
- If you correctly implemented this and still get a domain / target mismatch, be careful that your implemented response operator is not derived from an endomorphic operator, who has identical domain and target, which override class properties like `domain` and `method`
- If this line throws a domain / target mismatch error: `fl.jac.adjoint`: 