# Beginning

I decided to build a Discord bot that translates for multinational environments, since we live in one. The idea of turning it into my Hackathon entry came together with implementing brand text generation for an equally multinational setting (things like announcements, advertising messages and others), making the app a catalyst for languages and ideas in both an `Agentic Solution` and a `Brand Tool`.

## Early Stages

I started by setting up a Discord application, followed **[Discord’s own example](https://docs.discord.com/developers/quick-start/getting-started)** which made it super simple to move forward — didn’t waste much time here.

## The LLM Idea

Once the bot was running I started thinking about ways to handle the translation, and since I’d been working on other projects involving LLM development I decided to build one that would serve the translation process.

The idea was simple:

```
Bot message –API→ LLM
```

## Early AI usage (Claude Code) and LLM improvements

After that I started using the system Raptor pays for us to train scenarios with the LLM. That’s when I got the idea to add the text generation command, since the LLM was already working perfectly for translation.

I also set up some behaviors to streamline my development workflow, like improvement report generation and testing capabilities for the system (including for Docker — where, for example, I spun up an image just to make sure my LLM model was behaving as expected).

## Web Interface

I asked the agent to add a web interface using `React` because I wanted agencies that don’t use Discord to also be able to use the app as a `Brand Tool` — at this point it was no longer just a Discord bot but a full-on enterprise AI solution. I used ChatGPT’s and Claude’s interfaces as references, with a history system and an added option to choose the LLM’s personality (I built a login system for this just to stay within the Hackathon requirements).

## History

For the bot’s history system I wanted the behavior to be:

```
Discord bot
		}→ Shared history
Web interface
```

But to do that I’d need some way to make calls from the Discord bot be identified as the same user X from the web app’s backend — so I asked the LLM what the most correct approach would be and it set up the scheme to work more like:

```
Discord bot
	}Passes Discord user in header→ Shared history
Web interface
```

It also created a “profile” screen, and if the user had added their Discord username to their profile they would then be able to store history from both the Discord bot and the web app.

### SSE or Web Socket

I also wanted the history to update automatically, and since I’d never built a WebSocket system myself (only consumed one) I asked the AI agent to guide me through it — and it suggested SSE, since it would require fewer steps for the same result.