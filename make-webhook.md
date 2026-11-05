# Define a term with Claude and get it by email  

This tutorial builds a small AI workflow in Make. You type a term into a simple  
web page, Claude writes a short definition, and Make emails it to you.  

```
[Web page] → [Make: Custom webhook] → [Make: Anthropic Claude] → [Make: Send an email]
```

No programming is needed in Make. The web page is a single HTML file that you  
open in your browser.  

## What you need  

- A Make account (the free plan is enough for testing).  
- An Anthropic API key, created in the Claude Console at  
  console.anthropic.com.  
- An email account Make can send from, such as Gmail. Make's built-in Email  
  module works too.  
- A browser and a text editor for the HTML file.  

## How it works  

1. The web page sends the term and the chosen language to a webhook address in  
   Make.  
2. The webhook starts the scenario and passes the two values on.  
3. The Anthropic Claude module asks Claude for a definition.  
4. The email module sends Claude's answer to your inbox.  

The page doesn't wait for the definition. It only confirms that the term was  
sent, and the definition arrives by email a few seconds later.  

## Step 1: Create the scenario and the webhook  

1. In Make, open **Scenarios** and click **Create a new scenario**.  
2. Click the big plus, search for **Webhooks**, and choose **Custom webhook**.  
3. Click **Create a webhook**.  
4. Enter a name, for example `define-term`.  
5. Leave **API Key authentication** empty. An API key would have to be sent in  
   an HTTP header, and the browser can't send custom headers to Make without a  
   CORS preflight request that Make doesn't answer.  
6. Click **Save**.  

Make now shows the webhook address, for example:

```
https://hook.eu1.make.com/abc123yourhookid
```

Copy it. You'll need it for the web page in Step 2.  

Keep the module open: it says it is waiting for data. Make needs one real  
request to learn which fields the webhook receives.  

## Step 2: Create the web page  

Create a file called `define-term.html`, paste in the code below, and replace  
`YOUR_HOOK_ID` with the address from Step 1.  

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Define a term</title>
<style>
  :root {
    color-scheme: light;
    --bg: #EEF2F5; --surface: #FFFFFF; --ink: #1C2A39; --muted: #5B6B7A;
    --line: #C9D3DC; --accent: #2F6F62; --accent-ink: #FFFFFF; --error: #A23B3B;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      color-scheme: dark;
      --bg: #141B22; --surface: #1C252E; --ink: #E6ECF1; --muted: #93A3B2;
      --line: #33414E; --accent: #6FC2AE; --accent-ink: #10201C; --error: #E58B8B;
    }
  }
  * { box-sizing: border-box; }
  body {
    margin: 0; background: var(--bg); color: var(--ink);
    font: 1rem/1.5 system-ui, sans-serif;
  }
  main { max-width: 28rem; margin: 0 auto; padding: 4rem 1.25rem; }
  h1 { font: 600 2rem/1.2 Georgia, serif; margin: 0 0 1.5rem; }
  form { display: grid; grid-template-columns: 1fr auto; gap: .6rem; }
  input, select, button {
    font: inherit; padding: .7rem .85rem; border-radius: 6px;
    border: 1px solid var(--line); background: var(--surface); color: var(--ink);
  }
  button {
    grid-column: 1 / -1; border: 0; background: var(--accent);
    color: var(--accent-ink); font-weight: 600; cursor: pointer;
  }
  button:disabled { opacity: .6; cursor: progress; }
  :focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
  #status { min-height: 1.5rem; margin-top: 1rem; color: var(--muted); }
  #status.error { color: var(--error); }
</style>
</head>
<body>
<main>
  <h1>Define a term</h1>
  <form id="form">
    <input id="term" type="text" required autocomplete="off" placeholder="Term, e.g. webhook" aria-label="Term">
    <select id="language" aria-label="Language">
      <option value="Slovak">SK</option>
      <option value="English">EN</option>
    </select>
    <button id="submit" type="submit">Send</button>
  </form>
  <p id="status" role="status"></p>
</main>

<script>
  const WEBHOOK_URL = "https://hook.eu1.make.com/YOUR_HOOK_ID";

  const form = document.getElementById("form");
  const button = document.getElementById("submit");
  const statusEl = document.getElementById("status");

  form.addEventListener("submit", async (e) => {
    e.preventDefault();
    const term = document.getElementById("term").value.trim();
    const language = document.getElementById("language").value;
    if (!term) return;

    button.disabled = true;
    statusEl.className = "";
    statusEl.textContent = "Sending…";

    try {
      const res = await fetch(WEBHOOK_URL, {
        method: "POST",
        body: new URLSearchParams({ term, language })
      });
      if (!res.ok) throw new Error(`Status ${res.status}.`);
      statusEl.textContent = `Sent: ${term}. Check your email.`;
      form.reset();
    } catch (err) {
      statusEl.className = "error";
      statusEl.textContent = `Couldn’t send. ${err.message}`;
    } finally {
      button.disabled = false;
      document.getElementById("term").focus();
    }
  });
</script>
</body>
</html>
```

The page sends the data as form fields (`URLSearchParams`), not as JSON. This is  
deliberate: a form-encoded request is a "simple" request, so the browser sends  
it straight to Make. A JSON request would trigger a CORS preflight, which Make  
webhooks don't answer, and the request would fail.  

You can open the file by double-clicking it. If your browser blocks requests  
from local files, serve the folder instead:  

```bash
python3 -m http.server 8000
```

and open `http://localhost:8000/define-term.html`.  

## Step 3: Teach the webhook its fields  

With the webhook module still waiting for data, send one request. Either open  
the page, type a term, and click **Send**, or run this in a terminal:  

```bash
curl -X POST "https://hook.eu1.make.com/YOUR_HOOK_ID" -d "term=webhook&language=Slovak"
```

Make shows **Successfully determined**. From now on, the fields `term` and  
`language` are available in the following modules.  

If the module is no longer waiting, click it and choose **Redetermine data  
structure**, then send the request again.  

## Step 4: Add the Claude module  

1. Click the small plus to the right of the webhook and add **Anthropic Claude  
  → Simple Text Prompt**.  
2. Create a connection and paste your Anthropic API key.  
3. **Model:** choose a fast, inexpensive model such as Claude Haiku 5.5.  
4. **Max Tokens:** `1000` is plenty for a short definition.  
5. **Text Prompt:** paste the prompt below.  

```
You write short dictionary-style definitions for non-technical readers.

Term: [term]
Language of the definition: [language]

Rules:
- Write the definition in the language given above. If the language is Slovak, use correct Slovak grammar and diacritics.
- 2–3 plain sentences. Start with what the term is, then add one simple everyday example or use.
- Do not start with the term itself or with "It is" / "Je to". Begin directly with the explanation.
- No heading, no markdown, no bullet points, no quotation marks, no line breaks.
- Avoid jargon. If a technical word is unavoidable, explain it in a few words.
- If the term is unclear or has several meanings, define the most common one in technology and everyday use.
- If the input is not a real term or is empty, reply only with: Term not recognized.

Reply with the definition only.
```

Then replace the two placeholders with real fields. Delete `[term]`, keep the  
cursor there, and click **term** under Webhook in the mapping panel. Do the  
same for `[language]`. Mapped fields appear as small colored pills; typing  
`{{1.term}}` by hand doesn't work.  

Click **Save**.  

## Step 5: Add the email module  

1. Click the plus to the right of the Claude module and add **Gmail → Send an  
  email** (or **Email → Send an email**).  
2. Connect your email account.  
3. **To:** your own email address.  
4. **Subject:** `Definition: ` followed by the mapped **term** field.  
5. **Content:** map the text output of the Anthropic Claude module. You can add  
  the term above it, for example:  

```
[term]

[Claude's text output]
```

If you don't see Claude's output in the mapping panel, run the scenario once  
(Step 6) and come back. Make lists a module's outputs after it has run at least  
once.  

Click **Save**, then save the whole scenario with the **Save** button at the  
bottom.  

## Step 6: Test with Run once  

1. Click **Run once** at the bottom of the editor. The scenario now waits for  
  one request.  
2. Send a term from the web page.  
3. Each module shows a bubble with a number. Click it to see exactly what came  
  in and went out.  
4. Check your inbox for the email.  

The page says "Sent" as soon as Make accepts the request. The email takes a  
few seconds longer.  

## Step 7: Activate the scenario  

Run once handles only a single request. For the page to work any time:  

1. Switch the toggle at the top from **Inactive** to on.  
2. Keep the schedule at **Immediately as data arrives**.  

Every submitted term now runs the scenario straight away. Each run uses Make  
operations from your plan, so you can switch the scenario off when you don't  
need it.  

## Troubleshooting  

**The mapping panel shows no webhook fields.** The webhook hasn't received a  
request yet. Go back to Step 3.  

**The page says "Sent" but no email arrives.** The scenario is probably  
inactive. When it's inactive and not in Run once, Make still accepts the  
request but only queues it. Activate the scenario, or click Run once to process  
the queue.  

**The page shows "Couldn’t send. Failed to fetch".** Check the webhook URL in  
the HTML file, your internet connection, and that the request is still form-  
encoded. Adding a `Content-Type: application/json` header or an API-key header  
causes this error.  

**The Claude module fails.** Check the API key, that your Anthropic account  
has credit, and that a model is selected.  

**The definition is in the wrong language.** Check that **language** is mapped  
in the prompt, not typed as text.  

## Optional: a simple access check  

Anyone who knows the webhook address can send requests to it and use up your  
API credit. To make random requests stop early, add a shared token:  

1. In the HTML file, add a token to the request body:  

   ```js
   body: new URLSearchParams({ term, language, token: "kurz-2026" })
   ```

2. Send one request and redetermine the webhook's data structure so Make learns  
   the new field.  
3. Click the line between the webhook and the Claude module, choose **Set up a  
   filter**, and set the condition: **token** *Equal to* `kurz-2026`.  

Requests without the right token stop at the filter and never reach Claude. This  
isn't strong security, since the token is visible in the HTML file, but it's  
enough for a training demo.  

## Keep the webhook address private  

The webhook address works like a password. Don't publish the HTML file with your  
real address, for example in a public GitHub repository. If the address leaks,  
delete the webhook in Make, create a new one, and update the HTML file.  

## Ideas for next steps  

- Add a Google Docs **Insert a Paragraph to a Document** module after Claude to  
  collect all definitions in one glossary.  
- Add a Google Sheets **Add a Row** module to log each term, language, and date.  
- Change the prompt to translate, summarize, or classify text instead of  
  defining terms.  
