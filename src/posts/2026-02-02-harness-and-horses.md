---
title: "Building Systems That Build Systems"
date: 2026-02-02
---

Dear Friends,

I'm writing on Day One of the Fractal Tech Bootcamp, a 12-week software/AI engineering course. Today we got right into production work, as we were given two tasks to start the day: optimizing our Claude Code setups, and building a personal website.

The first task was supposed to be preparatory work; "improving our harnesses" for LLM-powered coding. The idea was that once we optimize how we work with Claude Code, the task of building a website will go more smoothly. So our actual deliverable was the deployed website (and this blog post), and the optimization work was the scaffolding.

---

### Harness Improvements

I've been working with Claude Code for a while now (pretty much as long as it has existed), and I've been working on a proprietary tool called [Threadlinking](https://github.com/thrialectics/threadlinking) for context engineering. There were a lot of other Claude Code optimization tasks that have been on my backburner, though, so today gave me the dedicated time to actually implement them. The goal was making the "vibecoding harness" smarter with better context preservation, automated verification, parallel workflows, and project-specific automation.

#### Context Management

The first problem was context loss between sessions. Every time I'd start fresh, Claude would re-explore the same files and I'd re-explain the same decisions. I already have my Threadlinking tool to address this, but I also created three skills to help:

`/session-summary` saves session context to threadlinking before clearing. It captures what changed, what decisions were made, and what's still in progress. `/project-map` generates architecture overviews — key files, module relationships, tech stack, entry points — so future sessions have structural context without re-exploration. `/handoff` creates detailed HANDOFF.md documents with recent changes, current state, and next steps, stored centrally in ~/.claude/handoffs/.

I also added a SessionEnd hook to settings.json that prompts for context saving when a session ends. It's prompt-type rather than automatic — asks before saving rather than just doing it.

#### Verification Feedback Loops

Earlier "Deep Research" into Claude Code optimizations had surfaced an insight: "Give Claude a way to verify its work" for 2-3x quality improvement. So I created verification tools.

`/verify` auto-detects the test framework (vitest, jest, playwright, pytest) and package manager, then runs the appropriate test command. It handles arguments like `--quick` for fast unit tests only, `--watch` for watch mode, and `--file <path>` for specific files.

`/design-qa` uses Playwright MCP for visual QA. `/design-qa screenshot <url>` captures page state. `/design-qa compare <url> <reference>` compares against a design file. `/design-qa review <url>` does a full design review checking layout, typography, color contrast, accessibility, and responsive behavior.

I also wrote check-tests.sh — a hook that runs after every file edit to hint about related test files. If I edit auth.ts and there's an auth.test.ts nearby, it reminds me.

#### Parallelization Infrastructure

For larger projects, I wanted to be able to run multiple Claude instances on different features simultaneously, a la Boris Cherny's personal setup. This required some infrastructure.

I installed tmux and configured it for Claude workflows — 50k line scrollback, mouse support, Alt+1-9 for window switching. Then I wrote claude-parallel.sh, a script that launches multiple Claude instances either in regular mode or with git worktrees for isolation. Each instance gets its own tmux window.

`/worktree` manages the git worktree lifecycle: create isolated workspaces, list active worktrees, clean up finished ones. Worktrees live in ~/.claude-worktrees/<project>/<name> to keep repos clean.

notify.sh handles macOS notifications with different sounds for completion, errors, and attention-needed states. I added shell aliases too: `cp5` launches 5 parallel Claudes, `cpw` launches with worktrees.

#### Project-Specific Automation

The final piece was making the harness adapt to each project. I replaced the simple check-tests.sh with project-hooks.sh — a conditional hook system that loads project-specific hooks from .claude/hooks.json. Now each project can define its own patterns: edit a file matching src/auth/** and it runs auth tests, edit manifest.json and it validates the JSON.

For workflow automation, I created four more skills. `/review` does code review — analyzes diffs for type safety, error handling, security issues, performance problems, and convention violations. `/tdd` guides test-driven development with state tracking through the red-green-refactor cycle. `/deploy` handles deployment workflows with pre-checks for production. `/claude-md` auto-maintains CLAUDE.md files by mining learnings from git commits and session context.

I tested the setup by exploring my projects folder for project-map candidates, then generated maps for three complex undocumented projects: my browser extension (127 files across UI/background/content/native modules), project-search (hybrid TypeScript+Python semantic search), and semantic-embedding-template (ML pipeline with 20+ analysis scripts). Then I added .claude/hooks.json to each with project-specific checks — TypeScript compilation for the TS projects, Python syntax checks, Node syntax validation for the JS project.

#### The Result

The full skill inventory now includes: `/session-summary`, `/project-map`, `/handoff`, `/verify`, `/design-qa`, `/worktree`, `/review`, `/tdd`, `/deploy`, `/claude-md` — plus the pre-existing `/minimalist-designer`, `/portfolio-builder`, and `/claudeconnect`.

The hooks system runs threadlinking file tracking and project-specific checks on every edit. Sessions can be summarized before clearing. Projects can define their own automation. Multiple Claudes can work in parallel on isolated worktrees. The harness got meaningfully smarter.

Although the real deliverable for the day was the website, I noticed that this scaffolding work was what I spent the majority of my time on, because it's orders of magnitude more interesting to me. I already know I'm more interested in the AI Engineering path than something like frontend/UX/web design work.

---

### Building the Website

I did finally get to building the website. I knew why I avoided it (I have always despised building websites, for whatever reason). I also knew up front that I get overwhelmed by major design choices, so I chose something minimalist in the style of Andrej Karpathy and Dustin Getz. Here's how I actually built it with Claude.

#### Skills and Context Gathering

The first thing I did was set up two custom Skills: `minimalist-designer` and `portfolio-builder`. These are essentially prompt templates that Claude Code can call to get specialized guidance. The minimalist-designer skill contains all the design principles from Karpathy's and Getz's sites. The portfolio-builder skill has workflows for analyzing repositories and turning them into presentable portfolio artifacts, which I didn't end up invoking today.

We then launched a couple of Explore agents in parallel to gather context: one examined my inspiration folder (which had PDFs of the Karpathy and Getz style guides), and another found existing content from my ByeMarianne and Threadlinking projects that could go in portfolio sections. This took maybe thirty seconds and returned reports on the design principles and available content/assets.

#### Tech Stack and Structure

For the actual site, Claude recommended Eleventy over pure HTML (because I wanted a blog with markdown posts) and over heavier frameworks like Next.js (because a portfolio site doesn't need React hydration). The site is about 120 lines of CSS, some Nunjucks templates, and markdown files for posts.

The file structure ended up being straightforward: a `src/` folder with templates in `_includes/`, data in `_data/`, CSS in `css/`, and markdown posts in `posts/`. Eleventy builds everything into a `_site/` folder that gets deployed.

#### Importing Substack Posts

For the blog content, I had nine existing posts from my [Substack](https://byemarianne.substack.com) in `~/Vault/Writing`. Claude wrote a Node.js script to parse the Substack export format — extracting titles, dates, and content from the markdown files, stripping out the Substack header boilerplate, and reformatting them with Eleventy-compatible frontmatter. The script also had to fix Substack's weird multi-line image syntax. Running it once converted all nine posts.

#### Design Details

I wanted a simple personal mark, so we added a sun glyph ☉ next to my name. Claude suggested putting it after the name rather than before so it would feel more like a signature seal than a corporate logo.

For the banner image, I described my aesthetic to Claude (dark backgrounds, copper linework, organic-meets-digital) based on images I'd collected in my inspiration folder. Claude wrote Midjourney prompts for five different concepts: mycelium networks, constellation threads, neural pathways, weaving loom threads, and tree ring cross-sections. I generated a few of each and picked the mycelial network — stippled nodes connected by organic threads on a dark background. It captures the "preserving context and connection" theme of my work.

#### Deployment

Deployment was very annoying, actually. I tried GitHub Pages first, but the Actions workflow kept getting stuck in a queue for fifteen-plus minutes. After talking with others at the end of the day, I think it's just that GitHub Pages was down. Without knowing this, and after multiple failed attempts at debugging environment permissions and branch configurations, Claude strongly suggested we switch to Netlify instead. One `netlify deploy --prod --dir=_site` command and it was live. I bought mariannedawson.dev through Hostinger and pointed the DNS records to Netlify's load balancer. Deployment took longer than actually building the site.

---

### Reflection

I kept pausing to work more on the "harness improvement" though. I would start working on the website and then notice something about the workflow that could use improvement, or I realized that I could turn what I was doing into a reusable skill, and it was like the website was work but the optimizations were play.

There's a major insight for me here, and it's related to the nature of AI tools like Claude Code. The possibility space is really expansive. For the first time in my life, I can leverage my systems thinking rapidly and build systems that build systems within an hour. I can create multiagent pipelines and concurrently design to control their tendency toward multiplicative bloat. Every time I work with Claude Code I end up thinking of three more things it could do, five more ways it could do it better, and I can genuinely spend hours doing just this.

It's really seductive to keep "improving the harness" when the process of doing that is intrinsically interesting and has immediate payoff. However, I run the risk of the illusion of productivity without actually shipping anything meaningful to anyone else in the world.

I'd bet I'm not alone here; I think anyone working with these tools will relate in one way or another. Particularly the systems thinkers. It invites you to build meta-systems and infrastructure and frameworks. Sometimes this kind of work is needed and has practical effect, but sometimes it's a way of avoiding the less sexy detail work of actually shipping.

I'm trying to think through this pattern and figure out which tendencies I can indulge vs. which to rein in. I want to be mindful of the things that actually need to exist in the world, and how to prioritize those, even when the systems-building is more fun for me.

I'm sure I will be continuing to update this blog with guides and recaps of the work I'm doing for the bootcamp, so stay tuned. Until then!

So Long,

Marianne
