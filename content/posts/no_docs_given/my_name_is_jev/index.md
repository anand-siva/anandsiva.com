+++
title = "My Name Is Jev?! Using Jev to See If My Cats Like Me and Answer Other Important Questions"
date = "2026-10-04T12:36:49-04:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Anand Siva"
authorTwitter = "" #do not include @
cover = "jev_coverimage.jpg"
tags = ["jev", "ai", "system-one", "devops", "golang"]
categories = ["devops"]
publications = ["No Docs Given"]
keywords = [
  "Jev tutorial",
  "TypeSafe AI",
  "System One model",
  "AI decision model",
  "structured AI output",
  "Jev Golang example"
]
description = "A hands-on introduction to Jev, TypeSafe's System One decision model, featuring cat approval ratings, structured AI answers, and a practical DevOps incident-response example in Go."
showFullContent = false
readingTime = true
hideComments = false
+++

The AI world moves very quickly nowadays. Every week I look, there is a new "frontier" model leading the "trust me, bro" benchmarks. I did not come up with that either; that is straight from the Fireship.io YouTube channel. It looks like AI token prices have been slashed for a while, but who knows anymore.

I read an interesting article the other day that said enterprises don't care which AI tool they use as long as it gets the job done. That is also how I feel in my personal life. While I am researching article topics or just asking general questions, I don't care if it is Claude or ChatGPT.

When I do use AI, though, I invariably use xhigh or high reasoning for questions like, "How do I know if my cats like me?" Is that the right tool for the job? Are those tokens worth it? This is a silly example of something that happens all the time in programming. There are tasks where I do not need "frontier" models, but that doesn't stop me from using them. Did I need to ask that using GPT-5.6 Sol? Hell yeah, I did!

## Asking ChatGPT the important questions


{{< image src="chatgpt-cat-likes-me.png" alt="ChatGPT answering the question: How do I know if my cats like me?" style="width:100%; max-width:100%;" >}}

That was a really open-ended, conversational-style answer. Pretty wordy as well. I could approach this problem another way. I already know how my cats act around me, so can I use that data to determine the same type of answer? Here is how my cats act around me:

1. Meow at me insistently
2. Attack my hand like it is a play toy
3. Occasionally hiss at me
4. Let me rub their bellies
5. Stare into my eyes as if looking into my soul

You know, normal cat stuff.

### Giving ChatGPT more context

Now let me ask ChatGPT with these parameters if my cats like me.

Prompt:

```text
This is how they act around me.

1. Meow at me insistently
2. Attack my hand like it is a play toy
3. Occasionally hiss at me
4. Let me rub their bellies
5. Stare into my eyes as if looking into my soul

Do my cats like me?
```

Answer:

```

😂 Yes. Based on this list, your cats very likely like you. They just appear to express love through a mixture of affection, violence, and psychological intimidation.

My translation of their behavior:

Insistent meowing: “HEY. HUMAN. I require your attention immediately.” Cats often develop specific meows for communicating with their humans.
Attacking your hand: If it's playful—no serious aggression, relaxed body language, stopping and starting—you're apparently an interactive cat toy. Try redirecting to a wand/kicker toy so your hands don't become the designated prey.
Occasional hissing: This doesn't mean they dislike you. It's usually “stop,” “I'm uncomfortable,” “you startled me,” or “I've had enough.” Respect the warning and give them space.
Letting you rub their bellies: This is a big trust signal. Many cats expose their stomach but absolutely do not permit touching. If yours genuinely relax while you rub their bellies, they're very comfortable with you.
The unbroken soul-piercing stare: Could mean affection/attention, curiosity, anticipation of food, or simply cat. If the body is relaxed, it's generally nothing concerning. Try slowly blinking at them and see whether they slow-blink back.
So I'd characterize the relationship as:

“We love this human. He feeds us, provides belly rubs, serves as prey when necessary, and occasionally needs to be hissed at for quality-control purposes.” 😹
```
Aww, that is sweet! I am loved by my cats! Let's gooo!

Also, lol at this statement: "They just appear to express love through a mixture of affection, violence, and psychological intimidation." Wild response.

This took some input and output tokens, and the answer was very conversational. What if I just needed a yes-or-no answer about whether they liked me? That is all.

## Asking Jev

That is where Jev comes in. I am going to show you the example first and then explain all the concepts after.

Let's ask Jev the same question: Do my cats like me? Jev has structured input and output, so here is how I would call it.

Jev has three types of primitives.


{{< image src="jev-primitives.png" alt="Jev primitive type selector showing Noul, Score, and Choice" style="width:100%; max-width:100%;" >}}

Let's use Noul for this.

I will explain the difference between State and Question later (partially because I currently have no idea what it means).

State:

```
My cats:
- Meow at me insistently
- Attack my hand like it is a play toy
- Occasionally hiss at me
- Let me rub their bellies
- Stare directly into my eyes
```

Question:

```
{
  "cats_like_me": {
    "type": "noul",
    "instructions": "Based on the provided state, do my cats' behaviors indicate that they like and trust me?",
    "criteria": {
      "true": "The behaviors overall indicate affection, trust, comfort, social bonding, or playful engagement.",
      "false": "The behaviors overall indicate fear, avoidance, hostility, persistent discomfort, or lack of trust."
    }
  }
}
```

Response:

{{< image src="jev-answer.png" alt="Jev response showing an 84 percent true and 16 percent false result for whether my cats like me" style="width:100%; max-width:100%;" >}}

Sweet! Looks like this robot is 84% sure my cats like me. Good enough!

### My cat Dexter approves

{{< image src="dexter-approves.jpg" alt="My cat Dexter holding my hand" style="width:100%; max-width:100%;" >}}

## What is Jev?

For these next two sections, I am going to have AI explain what Jev is because I do not fully understand it yet.

So what the hell is Jev? Jev is TypeSafe's flagship System One model. Unlike a chat model, it is not trying to have a conversation with you or generate a wall of text. You give it some state, ask it a narrow typed question, and it gives you a structured answer that your code can use directly.

That is the important difference. An LLM gives you strings meant for a human to read. Jev gives you a decision meant for software to act on. Instead of asking it to deeply analyze a complicated problem in one shot, you break the problem into small judgments and combine the answers in your own code.

According to [TypeSafe's introduction to Jev](https://docs.typesafe.ai/introduction), those judgments come in the three primitives we saw above: Choice, Score, and Noul.

## What does TypeSafe claim?

TypeSafe makes some pretty bold claims about Jev:

- **Structured answers instead of generated prose.** Your code gets typed values and probability distributions instead of a paragraph that it has to parse.
- **Calibrated probabilities.** The result tells your software how strongly the model leans toward an answer, so you can set a threshold and send uncertain cases to a human.
- **Fast, focused decisions.** Jev is designed for small judgments that a knowledgeable person could make quickly, not long chains of reasoning.
- **Questions run independently.** You can ask several questions about the same state in one request without one answer secretly becoming context for another.
- **Cheaper and faster than using a frontier LLM for the same kind of task.** TypeSafe's homepage currently claims Jev is **193.6x faster** and **444.6x cheaper** on its System One workflow comparison.
- **"Zero hallucinations."** That is TypeSafe's wording, and it is a huge claim. The practical idea is that Jev cannot invent a free-form response because every answer is constrained to the choices, levels, or yes/no judgment you defined. That does not mean every decision will be correct, so the returned probabilities still matter.

Basically, Jev is not trying to replace ChatGPT. It is trying to replace the part where we ask ChatGPT a narrow question, beg it to return valid JSON, and then hope our application can parse whatever comes back.

## My take

Reading through what Jev is trying to accomplish, I get it. We have all been there: we ask Claude a question, and it responds with a novel when sometimes we really just want a yes-or-no answer or a probability. Jev is not a code-generation tool like the other LLMs. Its superpower is classifying things, and it does this very, very fast. That cat question I asked before ran in a fraction of a second.

I am not sure it was 193.6x faster, since I do not have the time or patience to run Jev benchmarks. I love the "trust me, bro" claim that it will be 444.6x cheaper. This is very appealing to enterprises because of how expensive it is to run LLMs at scale. There are probably a huge number of questions we ask AI every day that can be answered by Jev. As a DevOps engineer, I spend a lot of time providing evidence and then asking an LLM if something is wrong or should be fixed. The claim of zero hallucinations is interesting as well, with TypeSafe basically saying that since Jev can't invent a free-form response, there are no hallucinations. I will be the judge of that the next time I ask my cat question and it returns `false`!

I can see Jev working in conjunction with LLMs to make a pretty robust system that can leverage the open-endedness of LLMs and the quick decision-making of Jev. I believe it can provide percentage-based answers as well. I will test this out in the next section.

Furthermore, instead of getting a long-winded, one-shot answer and going back and forth with an LLM, I can leverage Jev to make decisions on metric-based tasks in chunks.

## A real-world example

As much as I love using the power of AI to get my cats to like me, let's use a real-world example.

AWS has a service called DevOps Agent, which is used to help troubleshoot infrastructure problems. AWS usually has good products, but I don't trust it yet. Let's see how I can leverage Jev to answer some of my DevOps questions.

Let's say we have a performance issue with our database, and it is causing many application errors.

### Starting with a typical LLM

If this were a typical LLM, I would start the investigation like this:

Prompt:

```text
I need help troubleshooting my application.

Currently, 20% of requests across our application are returning 500 errors. Our database has a maximum of 3,000 connections, and there are currently 2,500 connections. It looks like 30% of the SQL queries are taking over 10 seconds. Latency is 4,200 ms.

Please analyze this situation.
```

Response:

```
Yes — these numbers strongly suggest your 500s are being driven by database saturation / request pile-up, not simply an application bug.
You have three signals reinforcing each other:
- 2,500 / 3,000 DB connections = 83% of the connection ceiling
- ~30% of SQL queries taking >10 seconds
- Application latency ~4.2 seconds
- 20% of requests returning 500
The dangerous part isn't just that you have 2,500 connections. It's that a large percentage of those connections may be occupied for a long time. That creates a queueing problem.
```

If I were building my own DevOps agent, I would probably have the code call a bunch of tools and collect numerous stats from my database. Then I would ask it to determine whether we should page the on-call engineer and use another tool call to fire off an alert.

Prompt:

```text
Should we page the on-call engineer?
```

Response:

```
Yes. With ~20% 500s, ~4.2s latency, 83% of DB connections in use, and 30% of SQL exceeding 10 seconds, I would page the on-call engineer now.
This is already a production-impacting incident, not just something to watch. The key reason is that you have both customer-visible errors and signs of database saturation, and those can deteriorate quickly if traffic stays steady.
I’d send something like:
We’re seeing ~20% 500s across the application. App latency is ~4.2s. DB connections are ~2,500/3,000, and ~30% of SQL queries are taking >10s. Suspect DB saturation / slow-query pileup. Paging for active incident investigation.

While they’re joining, keep gathering the top 500 exception types, active vs sleeping DB connections, DB CPU/IO/waits, and the most common slow SQL.
```

## Breaking it down with Jev

Database stats can easily be gathered, so the DevOps agent can collect a bunch of them first. That creates the state and basis for our follow-up questions.

Let's say these are the stats I have collected from the database:

```json
{
    "http_500_rate_percent": 20,
    "database_max_connections": 3000,
    "database_current_connections": 2500,
    "slow_sql_percent": 30,
    "slow_sql_threshold_seconds": 10,
    "application_latency_ms": 4200
}
```

I could make a simple Jev prompt that can ask multiple questions at once, but in a very deterministic way.

- **Noul:** Ask a yes-or-no question, such as whether the on-call engineer should be paged, and get back the probability that the answer is true.
- **Score:** Rate the incident on an ordered scale from normal operation to a critical outage.
- **Choice:** Pick exactly one action from a list, such as monitor, investigate, page, or escalate.

The judgment is still probabilistic, but the output is constrained to shapes and values my code already knows how to handle.

```bash
export TYPESAFE_API_KEY="your-api-key"

payload='{
  "model": "jev-latest",
  "state": {
    "http_500_rate_percent": 20,
    "database_max_connections": 3000,
    "database_current_connections": 2500,
    "slow_sql_percent": 30,
    "slow_sql_threshold_seconds": 10,
    "application_latency_ms": 4200
  },
  "questions": {
    "page_on_call": {
      "type": "noul",
      "instructions": "Should the on-call engineer be paged immediately?",
      "criteria": {
        "true": "There is significant production impact requiring immediate human intervention.",
        "false": "Immediate human intervention is not required."
      }
    },
    "database_is_likely_bottleneck": {
      "type": "noul",
      "instructions": "Is database pressure likely contributing significantly to the application incident?",
      "criteria": {
        "true": "The database metrics indicate substantial connection or query-performance pressure that could explain the application degradation.",
        "false": "The database metrics do not indicate enough pressure to plausibly explain the application degradation."
      }
    },
    "incident_severity": {
      "type": "score",
      "instructions": "Rate the severity of the current production incident.",
      "criteria": [
        "Normal operation",
        "Minor degradation",
        "Significant degradation requiring investigation",
        "Major incident requiring immediate response",
        "Critical outage"
      ]
    },
    "response": {
      "type": "choice",
      "instructions": "Choose the most appropriate immediate operational response.",
      "criteria": {
        "monitor": "Continue monitoring without immediate intervention.",
        "investigate": "Begin an investigation during normal operational response.",
        "page": "Page the on-call engineer immediately.",
        "escalate": "Page the on-call engineer and escalate as a major production incident."
      }
    }
  }
}'

curl -s https://api.typesafe.ai/v1/systemone \
  -H "Authorization: Bearer $TYPESAFE_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$payload" | jq
```

Response:


```json
{
  "model": "jev-1.13.0",
  "answers": {
    "page_on_call": {
      "type": "noul",
      "noul": 0.88
    },
    "database_is_likely_bottleneck": {
      "type": "noul",
      "noul": 0.88
    },
    "incident_severity": {
      "type": "score",
      "score": 3.0,
      "confidence": 0.96,
      "legend": {
        "0": "Normal operation",
        "1": "Minor degradation",
        "2": "Significant degradation requiring investigation",
        "3": "Major incident requiring immediate response",
        "4": "Critical outage"
      },
      "probabilities": {
        "0": 0.0,
        "1": 0.0,
        "2": 0.03,
        "3": 0.94,
        "4": 0.03
      }
    },
    "response": {
      "type": "choice",
      "choice": "escalate",
      "confidence": 0.76,
      "probabilities": {
        "investigate": 0.01,
        "monitor": 0.0,
        "page": 0.18,
        "escalate": 0.8099999999999999
      }
    }
  },
  "usage": {
    "input_tokens": 613,
    "output_tokens": 110
  }
}

```

Wow, that is quick, and it gave me exactly the type of output and decision I was looking for. Yes, it looks bad, and yes, let's escalate.

### Acting on the answer in Go

Here is a small example in Go:

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
)

type JevResponse struct {
	Answers map[string]struct {
		Type string  `json:"type"`
		Noul float64 `json:"noul"`
	} `json:"answers"`
}

func main() {
	apiKey := os.Getenv("TYPESAFE_API_KEY")
	pagingWebhook := os.Getenv("PAGING_WEBHOOK_URL")

	payload := map[string]any{
		"model": "jev-latest",
		"state": map[string]any{
			"http_500_rate_percent":          20,
			"database_max_connections":       3000,
			"database_current_connections":   2500,
			"slow_sql_percent":               30,
			"slow_sql_threshold_seconds":     10,
			"application_latency_ms":         4200,
		},
		"questions": map[string]any{
			"page_on_call": map[string]any{
				"type":         "noul",
				"instructions": "Should the on-call engineer be paged immediately?",
				"criteria": map[string]string{
					"true":  "There is significant production impact requiring immediate human intervention.",
					"false": "Immediate human intervention is not required.",
				},
			},
		},
	}

	body, _ := json.Marshal(payload)

	req, _ := http.NewRequest(
		"POST",
		"https://api.typesafe.ai/v1/systemone",
		bytes.NewReader(body),
	)

	req.Header.Set("Authorization", "Bearer "+apiKey)
	req.Header.Set("Content-Type", "application/json")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	responseBody, _ := io.ReadAll(resp.Body)

	var jev JevResponse
	if err := json.Unmarshal(responseBody, &jev); err != nil {
		panic(err)
	}

	probability := jev.Answers["page_on_call"].Noul

	fmt.Printf("Probability page is required: %.2f\n", probability)

	if probability >= 0.90 {
		fmt.Println("Paging on-call engineer...")
		pageOnCall(pagingWebhook, probability)
	}
}

func pageOnCall(webhook string, probability float64) {
	payload := map[string]any{
		"summary": "Application experiencing significant production degradation",
		"severity": "critical",
		"source":   "jev-demo",
		"details": map[string]any{
			"jev_page_probability": probability,
			"http_500_rate":        "20%",
			"latency_ms":           4200,
		},
	}

	body, _ := json.Marshal(payload)

	resp, err := http.Post(
		webhook,
		"application/json",
		bytes.NewReader(body),
	)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	fmt.Println("Page sent.")
}
```

What I love is this structured decision-making:

```go
	if probability >= 0.90 {
		fmt.Println("Paging on-call engineer...")
		pageOnCall(pagingWebhook, probability)
	}
```

This way I can feel like a programmer again and code (with AI) a program that can gather stats, run a few questions through Jev, and alert the on-call engineer. Realistically, if I wanted this to be a more proactive agent, I would take these results, feed them back into an LLM, and have it make some tool calls to start killing problematic queries or take another remediation action.

{{< image src="ironic.jpg" alt="Using AI to write AI API calls—isn't it ironic, don't you think?" style="width:100%; max-width:100%;" >}}

## Final thoughts

Only time will tell how effective Jev is since it is the new kid on the block. From the small example I have run, I can see the benefits of using Jev. I can also see the cost savings that can come from more compact context and structured answers. I also like how there is no memory, per se, and each question stands on its own.

Hopefully, that gives you a small example of Jev. I will be sure to stress-test it and look for ways to use both LLMs and Jev to make real applications that solve real-world problems. After writing these articles for a few months now, I think I will start creating some real applications that leverage everything I have learned. Stay tuned!
