---
layout: default
title:  "LLM-Based Coding Agents: Claude Code"
date:   2025-07-23 23:42:00 +0200
categories: llm programming
published: true
---

# Fun with LLM Agents

At work, I have been happily using [Amazon Q
Developer](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line.html)
at the command line for a while. Having been sceptical about LLM code agents, I
was pleasantly surprised how well the Q CLI integrates into my shell workflow.
Then I tried [Claude Code](https://claude.ai/code) for a private vibe coding
project and it confirmed my experience.

## My daily LLM use

After playing around with LLM agents a bit, I've found that I'm using them for
things that I would regularly have solved with a few Bash invocations chained
together.

Here's a typical example: I do testing of Amazon Linux across multiple EC2
instance types and regions. Now after some poking I need to tear down these
instances. This can easily be done with some Bash loops going through my active
EC2 regions and some invocations of the AWS CLI. If you have done that a couple
of times, most of the required commands live somewhere in your shell history
and are a `Ctrl+R` and some minor modifications away.

It turns out, LLM agents are pretty good at this too!

```
Find all EC2 instances across all available regions that have been launched in
the last 24 hours and terminate them.
```

is all it takes and my LLM agent will start invoking the right
commands and I can go ~~have a coffee~~solve another problem in the meanwhile.

The exciting part about LLM agents on the command line is that they have access
to the local machine. This is a huge step up from browser-based chat windows.
The agents can inspect (and modify) files on disk, invoke shell commands and
combine their outputs to perform their duties. I have so far successfully
used the Q CLI to
- write simple test scripts,
- debug why a QEMU VM didn't get an IP address via DHCP, and
- backport CVE patches to older Linux kernel releases.

## The Project

During my summer vacation, I decided to give LLM agents a try in a private
setting as well. I decided to try [Claude Code](https://claude.ai/code) and was
so hooked, that I was even willing to pay for the 20 EUR per month [Pro
subscription](https://www.anthropic.com/pricing). (I think this is as much as
I'll pay for private projects, the more exhaustive subscriptions will certainly
make sense if you're using it professionally.)

As a project, I wanted to write a tiny CLI program that displays weather
information about a given location. I knew I wanted to use Python as it's still
my go-to language for simple prototypes. I had recently read about
[Poetry](https://python-poetry.org) for Python building and packaging and so I
wanted to try this out. I also wanted to
[vibe-code](https://en.wikipedia.org/wiki/Vibe_coding) this, that is, I was
relying on Claude as my typing assistant.

The basic task here is quite simple: find an appropriate API service to query
weather data based on location, call that service via Python's `requests`
module, and print the response to the terminal.

I'm sure I could write that in about 20 lines of Python and be done with it.

Instead, I told Claude:

```
Write a terminal program that queries weather data for a given location and
prints the result to the terminal. Use Python as the programming language. Use
Poetry for building and dependency tracking.
```

Even the initial result was quite nice. Claude created a Poetry project. It
immediately decided to use my favourite CLI framework,
[Click](https://github.com/pallets/click) to build the actual CLI. Claude
researched weather APIs and told me to get a free subscription for
[OpenWeatherMap](https://openweathermap.org/). While I went through their signup
process, Claude generated a Python program wrapping the API call as expected.

``` text
# poetry run weather --city Dresden
Weather in Dresden, DE:
Temperature: 25.22°C (feels like 25.02°C)
Humidity: 47%
Conditions: Scattered Clouds
```

Now, the difference between my hypothetical 20 lines hack and the solution
generated by Claude is: Claude wrote more code. But that code is extensible,
split into reasonable components with APIs, which makes it much easier to work
with going forward. And Claude did all that in maybe 5 minutes.

### Extending the CLI

I then wanted to add more features. As others have [pointed out
before](https://manuel.kiessling.net/2025/03/31/how-seasoned-developers-can-achieve-great-results-with-ai-coding-agents/),
coding agents need proper guard rails in form of tests. This allows them to
make future modifications and stay on track instead of breaking existing
functionality. So I asked Claude to write tests and it did. This is a step I
repeated for new features and I eventually made it part of my prompts right
away.

After tests were in place, I asked Claude to add a location detection feature
and use that detected location instead of a CLI argument. Claude researched and
settled for a free geolocation IP detection service. This didn't quite work out
as this would end up using my internet provider's end point IP. So I asked
Claude to research platform-specific location services. It figured out how to
determine a computer's location in MacOS, Windows, and Linux, and provided me
an implementation that would actually detect my location.

Then I wanted some nice ASCII arts to go along with the weather data. I
proposed Claude to render an ASCII representation of the weather next to the
program output. Claude then figured out from OpenWeatherData's API
documentation what weather codes existed and derived a set of emoji characters
to use.

With this, we now have a nice output:

```
# poetry run weather --here
Weather in Mariannelund, SE:              │      *   *
Temperature: 18.37°C (feels like 18.74°C) │    *
Humidity: 95%                             │        🌙
Conditions: Clear Sky                     │    *        *
                                          │      *   *
```

## Prompt engineering matters

Overall, I'm happy with the outcome. Vibe coding seems to work once you get the
hang of writing appropriate prompts. Three things to keep in mind emerged:

**Specific Prompts**: The more complex your ask is, the more detailed your
prompt needs to be. You can expect Claude to write a simple API wrapper in your
language of choice (as I did). But once I got to location detection, Claude at
first chose the wrong API because I wasn't specific enough and the language
model doesn't "understand" the differences between geolocated IPs and system
location.

**Leverage tests and linters**: Claude will get lost between sessions and when
your context runs out. Even if you carefully manage your `Claude.md` (a great
feature where Claude keeps a diary of project-specific knowledge), at some
point the generated code will have flaws. Make sure to generate tests for all
APIs and functionality. Force Claude to use linters periodically. Be adamant
that test or linting failures are bad and need to be fixed.

I actually struggled a while. When I added weather art, lots of CLI tests were
failing as the output had changed. Initially, Claude ran the tests and didn't
think much of the failures. It even told me that these were just because our
output recently changed and not to worry about them. Eventually, I convinced
Claude to add this section to its `Claude.md` file and that did the trick:

``` markdown
## Development Best Practices

- Linting: after every code change, run 
  `poetry run black . && poetry run isort && poetry run flake8 && poetry run mypy src/`
  and fix all reported issues
- Testing: after every code change, run `poetry run pytest` and fix all
  failures
```

**Claude writes lots of code**: Claude's code is verbose. It will contain
unnecessary comments (I didn't manage to untrain this), duplicate code, and
missing abstractions. I have found it helpful to use a prompt like this every
few commits:
```
Analyze the architecture and code. Find opportunities to reduce code size and
improve clarity.
```
This will trigger useful refactoring of your code base. Make sure to have
sufficient tests in place as to not break existing functionality.

## The Elephant in the Room

Somewhere on the web I read a description describing LLM-based coding agents as
_"very senior programmers, but very junior developers"._ I've found this
characterization to be very true. At a language/framework/library level, LLM
agents are as smart as someone that has read all the documentation. And I
really mean **all** the documentation across all the internet.

However, the agent on its own will still "behave" very junior.  You constantly
need to re-educate the agent about specific rules. It does not ever learn about
coding style or development best practices. Be prepared to repetitively ask
your coding agent to add/fix tests and update your project's `README.md`.

Claude Code provides means to at least partially [handle this lack of
memory](https://docs.anthropic.com/en/docs/claude-code/memory): you can have
per-system and per-project `Claude.md` files that will be added to the LLM
agent's context every time you use it.

### An unpleasant surprise

About an hour after a Claude Code session on my CLI project. I received this
email:

<div align="center">
<img src="/assets/images/gitguardian.png" alt="I leaked an API token" width="75%">
</div>

A click on the provided link validated that I had indeed [checked in my
plain-text API key into the GitHub
repository](https://github.com/bjoernd/weathercli/blob/c9435cf2f5154ffc557da49cdab744d5e84e3e5d/weather_debug.log#L16).

**How could this happen?**

1. In an earlier feature session I had asked Claude to [add debug
   logging](https://github.com/bjoernd/weathercli/commit/7f9117df8b8b39bb159ae4bd5fe7fb23d43fbd44)
   to the program, which it did by adding debug logs through Python's debug
   facility and also into a local log file.
2. I even had the presence of mind to ask Claude to ignore that log file in
   order to not push it to GitHub. However, Claude screwed up this [simple
   modification of`.gitignore`](https://github.com/bjoernd/weathercli/commit/249cae1ed41db138d71886361e5a2a0b4dd60ce2)
   and I did not properly validate this step.
3. As a consequence, a [later, unrelated
   commit](https://github.com/bjoernd/weathercli/commit/c9435cf2f5154ffc557da49cdab744d5e84e3e5d)
   unexpectedly added a local copy of the debug log file to the git repository.
   And I once again did not properly validate the commit.

Note that Claude actively added debug logs leaking a plaintext key here.
Wherever your work, I'm sure you will have periodic security training teaching
you that these kind of logs are an absolute no-go.

Luckily, leaking API keys via git repositories is such a common occurrence that
GitHub has actual scanners for those. I immediately deactivated the leaked API
key in OpenWeatherData, removed the log file from the repository, and fixed the
`.gitignore` miss. I then even had to go one step further and [disable debug
logs for Python's
`urllib3`](https://github.com/bjoernd/weathercli/commit/f571767dad1c582915fd02270a96ac279b52d498)
as these were still showing API calls with the full URL including the API key.

## Summary

I'm quite happy with the result of this toy project. If you're interested, you
can find the full [project repository on
GitHub](https://github.com/bjoernd/weathercli).

Claude Code did 99% of the typing in this project. The resulting code is much
better designed and tested than I would have done this myself for a side
project. Using an LLM-based agent as a replacement for your keyboard certainly
helps and gives you quite nice results.

Be aware of the caveats, though: even if the LLM agent writes code and all the
tests pass, you are still responsible for the code. I wrote earlier that Claude
added API key logging and commited it. But in the end it was _I_ who was
supposed to validate the code does the right thing and doesn't violate security
expectations. Coding agents are tools, but you are still the builder wielding
them.
