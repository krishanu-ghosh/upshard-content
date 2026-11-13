---
title: LLMs for Data Scientists
description: Using large language models as an analysis tool, for SQL generation, summarization, and sense-checking results.
roadmap: ai-data-scientist
node: llms-for-data-scientists
group: shipping-insights
video:
  provider: youtube
  id: "dvFDxwlT2e8"
  start: 0
  end: null
creator:
  name: Tina Huang
  channel_url: https://www.youtube.com/@TinaHuang1
duration_seconds: 1308
level: intermediate
prerequisites: [ai-assisted-data-science-workflows, taking-models-to-production]
takeaways:
  - "An LLM is useful for text-to-SQL, summarizing results in plain language, and drafting exploratory code, but it has no privileged access to your schema or your data's actual distribution unless you give it both explicitly."
  - "Text-to-SQL fails most often on ambiguity the model can't see: an unqualified column name that exists in two joined tables, or a business term like \"active user\" that has no single agreed definition."
  - Asking an LLM to summarize a result table is asking it to also interpret it, and interpretation can smuggle in a causal or comparative claim your data doesn't support, so the summary needs the same scrutiny as the analysis itself.
  - Giving an LLM a schema, sample rows, and known edge cases before asking for SQL or an explanation produces dramatically better results than asking it to guess at your data's structure.
check_questions:
  - q: Why does an LLM generating SQL from a plain-English question often fail on ambiguous column names?
    a: Because the model has no view of your actual schema unless you provide it, so when two joined tables share a column name like status, it has to guess which one you meant, and it will guess confidently rather than ask.
  - q: What risk does asking an LLM to "summarize" a results table introduce that asking it to "describe" the table does not?
    a: Summarizing invites interpretation, such as attributing a change to a cause, which can state a claim your data doesn't actually support, while describing just restates the numbers without adding unsupported claims.
  - q: What's the single most effective way to improve the quality of LLM-generated SQL or analysis?
    a: Providing the schema, a few real sample rows, and any known data quirks up front, rather than relying on the model to infer the structure of your data from the question alone.
deeper:
  - label: "AI & Text to SQL: How LLMs & Schema Power Data Analytics"
    url: https://www.youtube.com/watch?v=S5ezVVJhQmE
  - label: How to use generative AI to make data science better
    url: https://www.youtube.com/watch?v=Lov1pI6kNxA
related_formulas: []
---

## A tool, not a teammate who knows your data

This lesson is about LLMs as something a data scientist uses, not about how LLMs work internally. The useful mental model is: an LLM is extremely good at producing fluent, plausible-sounding language and code from a description, and it has zero built-in knowledge of your company's database, your team's definition of "active user," or the one join that always needs a `DISTINCT` to avoid duplicate rows. Every bit of that context has to come from you. The gap between "worked great in the demo" and "produced a wrong number in production" is almost always a gap in context the model wasn't given, not a limitation of the model's language ability.

## Text-to-SQL and where it breaks

Asking an LLM "what were our top 10 customers by revenue last quarter" and getting back a runnable SQL query is one of the most immediately useful applications for a data scientist, especially across an unfamiliar schema. It genuinely saves time looking up table names and join keys. But it breaks down in predictable ways:

- **Ambiguous columns.** If `orders` and `refunds` both have a `status` column, and you ask for "orders with a pending status," the model has to guess which table's `status` you mean, or whether you mean a joined, combined status. It will often guess and move on rather than flag the ambiguity, because a fluent guess is what it's optimized to produce.
- **Undefined business terms.** "Top customers by revenue" sounds precise but isn't: does revenue mean gross or net of refunds, booked or recognized, including or excluding tax? An LLM will pick a reasonable-sounding default, like summing a `total_amount` column, which may not match the definition finance actually uses.
- **Hidden duplication.** A join that fans out rows, such as joining orders to a line-items table without aggregating first, inflates a sum silently. The query runs, returns a number, and that number is wrong by exactly the duplication factor.

The fix for all three is the same: give the model your schema (table and column names, with types), a few real sample rows so it sees what the data actually looks like, and a one-line definition of any ambiguous business term before you ask the question. A prompt that says "here is the schema, here are 5 sample rows from `orders`, revenue means `total_amount` minus `refund_amount`, write SQL for top 10 customers by revenue last quarter" will produce a dramatically more reliable query than the bare natural-language question.

## Summarizing and interpreting results

LLMs are also useful for turning a results table into plain-English sentences, which is a real time-saver when you're drafting the first pass of a report. But there's a meaningful difference between *describing* a result and *interpreting* it, and the second is riskier.

Describing is safe: "Revenue in March was $420K, up from $390K in February." That's a restatement of numbers already in the table.

Interpreting is where unsupported claims creep in: "Revenue increased in March because of the new pricing page." Unless your data actually supports a causal link, an LLM asked to "summarize what happened" will often supply a plausible-sounding reason anyway, because a narrative is more fluent than a bare number, and fluency is what the model is good at producing. If you ask an LLM to summarize a result, read the summary specifically for causal or comparative language ("because," "driven by," "outperformed") and check whether your analysis actually established that, or whether the model added a story on top of the numbers. The same scrutiny you'd apply to a human analyst's write-up applies here, arguably more so, because the model's confidence doesn't track its actual certainty.

## A worked habit: schema-first prompting

A simple routine covers most of the gap. Before asking an LLM for SQL, code, or a summary, paste in (1) the relevant table schemas, (2) three to five real rows, including one with a null or an edge case if one exists, and (3) a one-sentence definition of any term in your question that isn't a literal column name. This takes under a minute and converts the model from guessing about your data to reasoning about data you've actually shown it. It's the single most effective habit for using LLMs well in a data science workflow — more impactful than any particular prompting trick, because most failures trace back to the model inferring structure it was never given, not to the model reasoning badly about structure it was given.

## Where this fits with coding assistants

This overlaps with, but isn't the same as, using an AI coding assistant to write analysis code. The difference is the artifact: a coding assistant produces a script you'll run and iterate on yourself, while an LLM used for SQL generation or summarization often produces something closer to a final answer that gets read or executed once. That shorter feedback loop is exactly why schema-first context and skeptical reading of summaries matter more here, not less: there's less chance to notice a wrong assumption before it reaches someone's inbox.
