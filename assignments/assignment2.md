# Assignment 2: Food Tracker

## Problem Statement

**Food Tracker**

**Problem Domain:**
Food is a large part of my life, but I am uneducated about which foods are healthy or how foods affect me. When I was in Amsterdam for the summer, I didn’t know which foods to cook or what specific flavor each ingredient added to a meal. I want different ways to learn how each food affects me and get inspiration for new meals.

**Core Problem:**
Connecting foods to personal outcomes.
People react differently to the same foods. Users struggle to understand which ingredients or meals worsen symptoms or sap energy, because logs are tedious and feedback is generic. The core difficulty is turning everyday meal notes and quick check-ins into simple, trustworthy insights about personal responses—and then acting on them with safer swaps.

**Stakeholder List:**
- **Health-Conscious Individual:** Wants to understand how food affects their body and well-being.
- **Healthcare Provider/Nutritionist:** Uses data to advise patients/clients on dietary choices.
- **Food Product Companies:** Can use anonymized data to improve or market healthier products.
- **Fitness Influencers:** Can promote certain foods that show health benefits.

**Evidence and Comparables:**
- People react differently to the same foods. Studies show that everyone’s body responds in unique ways to meals, so personalized diet advice can help people feel better and stay healthier. ([PubMed](https://pubmed.ncbi.nlm.nih.gov/26590418), [Cell](https://www.cell.com/fulltext/S0092-8674%2815%2901481-6))
- New research in 2025 links differences in blood sugar responses to genetics and gut bacteria, proving that personalized nutrition is important for health. ([Nature](https://www.nature.com/articles/s41591-025-03719-2))
- The PREDICT program, which powers the ZOE app, uses large studies to show how people’s bodies react to food. This evidence supports the idea of apps that track personal food effects. ([Zoe](https://zoe.com/our-studies?srsltid=AfmBOop5eX7LzTIl7B-brsQ2XDegDeYvKw_V7lUboiMkHjVNHDi9PvAD))
- Mobile health apps are starting to predict when people are vulnerable to blood sugar spikes and send timely advice. This shows that AI-driven nutrition insights are possible and useful. ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC12271334))
- Reviews of nutrition apps argue that machine learning and personalization can help people more than generic diet advice. This means smarter, individualized tools are needed. ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC12350580))
- Gut sensitivity and IBS are common worldwide. Studies show that many people need to track how foods affect their symptoms, so there’s a large potential user base for personalized food tracking. ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC10354571), [Frontiers](https://www.frontiersin.org/journals/medicine/articles/10.3389/fmed.2022.922063/full), [JNM Journal](https://www.jnmjournal.org/journal/view.html?doi=10.5056%2Fjnm22037))
- Standard calorie tracking apps often have accuracy problems and people stop using them over time. Studies show that smarter, more engaging tools are needed to keep users motivated. ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC4422872))
- Counting calories can be misleading. Dietitians and health articles say that absorption varies and apps can sometimes encourage unhealthy eating habits, so it’s better to focus on how foods actually affect you. ([SELF](https://www.self.com/story/counting-calories-not-necessary-weight-loss), [EatingWell](https://www.eatingwell.com/article/8065441/problem-with-calorie-tracking-apps-dietitian-instagram))
- MyFitnessPal is a popular app for tracking calories and macros, but studies question its accuracy for some people. There’s a gap for apps that track the real effects of foods over time. ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11221275))
- Cronometer is good for tracking micronutrients, but it relies on users to log everything and doesn’t analyze how foods affect symptoms or health. A new app could close this gap. ([Cronometer](https://cronometer.com/index.html), [Reddit](https://www.reddit.com/r/cronometer/comments/1jakfe7/how_accurate_do_you_find_the_apps_micronutrient))

---

## Application Pitch

**Name:**
Forkcast

**Motivation:**
Know which foods help—or hurt—you, without obsessive tracking.

### 1. Effortless Capture
- **What it is:**
	A super-light way to log meals and quick check-ins. Snap a photo or speak a note; the app extracts likely ingredients and time. Add a 10-second check-in for energy, mood, or gut comfort.
- **Why it helps:**
	The core problem is connecting foods to personal outcomes; that requires enough data without burnout. Low-friction capture increases logging consistency, which is the foundation for any meaningful insight.
- **Stakeholder impact:**
	- Health-Conscious Individual: spends less time tracking, more time learning.
	- Healthcare Provider/Nutritionist (with consent): gets a cleaner longitudinal record to discuss in sessions.


### 2. Coach-Assistant (Combined)
- **What it is:**
	A single experience that both (a) finds your food→outcome patterns and proposes swaps, and (b) answers your questions (“Do late-night fried meals hurt my sleep?”) using your own data and insights.
- **Why it helps:**
	Turns "know" into "do," and gives personalized answers instead of generic advice.
- **Stakeholder impact:**
	- Health-Conscious Individual: gets concrete next steps and fast explanations.
	- Healthcare Provider/Nutritionist: sees better adherence and can tailor advice.

### 3. Weekly Progress
- **What it is:**
	A lightweight weekly summary of trends (top helpful/harmful signals, outcome trajectories) that you can skim in under a minute.
- **Why it helps:**
	Reinforces wins, prevents overreacting to one-off blips, and keeps motivation up.
- **Stakeholder impact:**
	- Health-Conscious Individual: stays on track and sees progress.
	- Healthcare Provider/Nutritionist: can review progress quickly and support ongoing improvement.

## Concept Design
### Concept: MealLog [User, FoodItem]

**purpose**
Capture meals quickly with minimal friction.

**principle**
A user records a meal with time, items, and an optional note; meals can be edited or deleted. This concept stores facts only.

**state**
- A set of Meals with:
	- owner: User
	- at: Time
	- items: Set(FoodItem)
	- notes: String (optional)
	- status: {active, deleted}

**actions**
- `submit(owner: User, at: Time, items: Set(FoodItem), notes?: String): (meal: Meal)`
	- **requires:** owner exists and items is nonempty
	- **effects:** create an active meal and return it
- `edit(meal: Meal, items?: Set(FoodItem), notes?: String)`
	- **requires:** meal exists and caller is the owner and status is active
	- **effects:** update provided fields
- `delete(meal: Meal)`
	- **requires:** meal exists and caller is the owner and status is active
	- **effects:** set status = deleted

---

### Concept: QuickCheckIns [User, Metric]

**purpose**
Record simple self-reports to correlate with recent meals.

**principle**
A user logs outcomes such as energy, mood, or gut comfort as numeric values at times; this concept stores facts only.

**state**
- A set of CheckIns with:
	- owner: User
	- at: Time
	- metric: Metric (e.g., energy, mood, gut_discomfort)
	- value: Number

**actions**
- `record(owner: User, at: Time, metric: Metric, value: Number): (checkIn: CheckIn)`
	- **requires:** owner exists and metric is allowed
	- **effects:** create a check-in and return it
- `defineMetric(metric: Metric)`
	- **effects:** register a new allowed metric

- `edit(checkIn: CheckIn, metric?: Metric, value?: Number)`
	- **requires:** checkIn exists and caller is the owner
	- **effects:** update provided fields (metric and/or value)

### Concept: InsightMining [User, Signal, Metric]

**purpose**
Turn observations into per-user insights and weekly summaries.

**principle**
The system ingests observations (signals and metric values over time), analyzes them to publish insights with effect and confidence, and can summarize trends over a period.

**state**
- A set of Observations with:
	- owner: User
	- at: Time
	- signals: Set(Signal) (tags like fried, dairy, late_night)
	- metric: Metric
	- value: Number
- A set of Insights with:
	- owner: User
	- signals: Set(Signal)
	- metric: Metric
	- effect: Number (positive or negative)
	- confidence: Number (0..1)
	- active: Flag
- A set of Reports with:
	- owner: User
	- period: String (e.g., "week")
	- generatedAt: Time
	- topHelpful: Set(Signal) (optional)
	- topHarmful: Set(Signal) (optional)
	- metricTrends: Set(Metric, Number) (optional)

**actions**
- `ingest(owner: User, at: Time, signals: Set(Signal), metric: Metric, value: Number)`
	- **effects:** add an observation for owner
- `analyze(owner: User, window: Hours)`
	- **requires:** exists observation for owner within window
	- **effects:** compute insights from those observations; create or update Insights with effect and confidence
- `summarize(owner: User, period: String): (report: Report)`
	- **requires:** exists observation for owner within the period
	- **effects:** compute a trend summary; create Report; return it
- `deactivate(requester: User, owner: User, signals: Set(Signal), metric: Metric)`
	- **requires:** insight exists for (owner, signals, metric) and requester = owner
	- **effects:** set that insight’s active = false

*Note: ingest and analyze/summarize may be triggered by system syncs; deactivate is an end-user action guarded by ownership.*

---

### Concept: SwapSuggestions [User, Signal, Alternative]

**purpose**
Propose practical alternatives when an insight indicates risk.

**principle**
When a signal is negatively associated with an outcome at sufficient confidence, the system proposes safer alternatives; the user may accept a proposal.

**state**
- A set of Proposals with:
	- owner: User
	- risky: Set(Signal)
	- alternatives: Set(Alternative)
	- rationale: String
	- accepted: Flag

**actions**
- `propose(owner: User, risky: Set(Signal), alternatives: Set(Alternative), rationale: String)`
	- **effects:** add a proposal for owner
- `accept(requester: User, owner: User, risky: Set(Signal), alternatives: Set(Alternative))`
	- **requires:** matching proposal exists for owner and requester = owner
	- **effects:** set accepted = true

---

### Concept: PersonalQA [User, Fact, Question, Answer, Time]

**purpose**
Answer a user’s food questions using their own meals, check-ins, and insights.

**principle**
The assistant maintains a private fact base of normalized statements (from meals, check-ins, insights, behavior changes) and answers questions by citing those facts.

**state**
- A set of Facts with:
	- owner: User
	- at: Time
	- content: Fact (e.g., “late_night + fried linked to lower energy (conf 0.82)”)
	- source: String ("meal", "check_in", "insight", "behavior")
- A set of QAs with:
	- owner: User
	- question: Question
	- answer: Answer
	- citedFacts: Set(Fact)

**actions**
- `ingestFact(owner: User, at: Time, content: Fact, source: String)`
	- **effects:** add a fact
- `forgetFact(requester: User, owner: User, fact: Fact)`
	- **requires:** fact exists for owner and requester = owner
	- **effects:** remove the fact
- `ask(requester: User, question: Question): (answer: Answer, citedFacts: Set(Fact))`
	- **requires:** requester exists
	- **effects:** produce an answer derived from requester’s Facts; store QA with owner = requester; return answer with citedFacts

### Essential Synchronizations

**mealSubmittedPipeline**

*When*: `MealLog.submit(requester, at, items, notes?)`

*Then*:
- `InsightMining.ingest(owner: requester, at: at, signals: items, metric: "meal_event", value: 1)`
- `PersonalQA.ingestFact(owner: requester, at: at, content: formatMealFact(items, notes?), source: "meal")`
- `InsightMining.analyze(owner: requester, window: 48h)`

**checkInRecordedPipeline**

*When*: `QuickCheckIns.record(requester, at, metric, value)`

*Then*:
- `InsightMining.ingest(owner: requester, at: at, signals: {}, metric: metric, value: value)`
- `PersonalQA.ingestFact(owner: requester, at: at, content: formatCheckInFact(metric, value), source: "check_in")`
- `InsightMining.analyze(owner: requester, window: 48h)`

**mealChangedRecompute**

*When*: `MealLog.edit(requester, meal, items?, notes?)` or `MealLog.delete(requester, meal)`

*Then*:
- `InsightMining.analyze(owner: meal.owner, window: 48h)`
- `PersonalQA.ingestFact(owner: meal.owner, at: Now(), content: formatMealChangeFact(meal, items?, notes?), source: "meal")`

**analysisToProposals**

*When*: `InsightMining.analyze(owner, window)`

*Then*:
- `SwapSuggestions.propose(owner: owner, risky: selectRiskySignals(owner), alternatives: selectAlternatives(owner), rationale: buildRationale(owner))`
- `PersonalQA.ingestFact(owner: owner, at: Now(), content: formatInsightFact(owner, window), source: "insight")`

**behaviorChangeReanalyze**

*When*: `SwapSuggestions.accept(requester, owner, risky, alternatives)`

*Then*:
- `InsightMining.analyze(owner: owner, window: 48h)`
- `PersonalQA.ingestFact(owner: owner, at: Now(), content: formatBehaviorFact(risky, alternatives), source: "behavior")`

**summarizeAndRecord**

*When*: `InsightMining.summarize(owner, period): (report)`

*Then*:
- `PersonalQA.ingestFact(owner: owner, at: report.generatedAt, content: summarizeReportFact(report), source: "insight")`

Brief Note: In the app, MealLog and QuickCheckIns are the fact collectors (users record meals and quick outcomes), InsightMining is the insight engine that ingests abstract observations (signals plus metric values), computes per-user insights and weekly summaries, SwapSuggestions is the action layer that proposes safer alternatives when an insight indicates risk and records acceptance, and PersonalQA is the explanation layer that stores normalized facts (from meals, outcomes, insights, behavior changes) and answers a user’s questions with citations; these concepts remain independent and are composed only via syncs (e.g., a submitted meal is tagged into signals → ingested → analyzed → may trigger a proposal, while key statements also flow into PersonalQA). Access control is uniform without a separate authentication concept: every user-mutating action takes a `requester: User` and guards `requester = owner`; in practice **User** is an opaque **AppUserId** supplied by the app’s local profile (single-user mode can fix this to a constant “Self”; multi-profile mode selects the active profile), and system actions like `analyze`/`summarize` run only for the owner passed via syncs. Generic types are instantiated concretely: **FoodItem** → canonical `FoodId` (MVP normalized strings; later swappable to UPC/OpenFoodFacts/USDA IDs), **Signal** → curated `TagId` (e.g., fried, dairy, spicy, caffeine, high\_sugar, whole\_grain, plus context tags like late\_night) derived from FoodItem and Time in sync adapters, **Metric** → controlled `MetricId` (energy, mood, gut\_discomfort; optionally sleep\_quality or glucose) with scales defined outside concepts, **Alternative** → `AlternativeId` from a small swap library (e.g., grilled\_instead\_of\_fried, lactose\_free\_instead\_of\_dairy), **Question/Answer** → plain strings, **Fact** → structured serialized strings (e.g., “kind\:insight; signals=fried,dairy; metric=gut\_discomfort; effect=-0.42; conf=0.82”) with formatting/parsing in adapters, **Time** → ISO-8601 UTC and **Hours** → numeric duration, and InsightMining reports expose semantic fields (sets of Signals and (MetricId, value) pairs) rather than generic maps so they’re easy to cite and evolve.

## User Journey:

Maya, a sophomore juggling classes and a campus job, keeps crashing mid-afternoon and wants one small change that will steady her energy. On Monday at lunch she opens the app to the Meal Log home, which shows Today and Yesterday at a glance (Sketch 1). She taps Log Meal, sees the search bar and camera option (Sketch 2), and chooses the camera. She snaps her bowl (Sketch 3); the app recognizes key items and tags them, and the meal appears back on the home list (Sketch 1). This low-friction capture is the habit she can actually keep.

By 2:30 p.m. she feels a slump and opens Health Check-In (Sketch 4). A lightweight survey pops up (Sketch 5) with sliders for energy, mood, and gut comfort; she submits in seconds. The app quietly analyzes her recent meals and outcomes.
A moment later, an AI suggestion pops up (Sketch 6): “Fried dinners are linked to lower next-day energy. Try grilled instead.” It offers a one-tap swap; she accepts. That night she’s curious and opens the AI Assistant page (Sketch 7) to ask, “Do late dinners matter too?” The assistant answers in plain language, citing her own logs and check-ins, and recommends finishing dinner before 8:30 p.m., giving her both the “why” and a concrete next step.

On Sunday, Maya checks the Weekly Summary (Sketch 8). It highlights “Top harmful: fried dinners / late dinners,” “Top helpful: grilled dinners,” and shows her afternoon energy trend rising by about two points. This is an immediate outcome she couldn’t get from generic nutrition advice.

## UI Sketches

Below are the UI sketches referenced in the user journey and concept design:

**Sketch 1: Log Meal**
![Sketch 1: Log Meal](../assets/UI%20Sketch-1.jpg)

**Sketch 2: Check-In**
![Sketch 2: Check-In](../assets/UI%20Sketch-2.jpg)

**Sketch 3: Q&A + Insight**
![Sketch 3: Q&A + Insight](../assets/UI%20Sketch-3.jpg)

**Sketch 4: Weekly Summary**
![Sketch 4: Weekly Summary](../assets/UI%20Sketch-4.jpg)

**Sketch 5: Swap Suggestions**
![Sketch 5: Swap Suggestions](../assets/UI%20Sketch-5.jpg)

**Sketch 6: Progress Trends**
![Sketch 6: Progress Trends](../assets/UI%20Sketch-6.jpg)

**Sketch 7: Fact Details**
![Sketch 7: Fact Details](../assets/UI%20Sketch-7.jpg)

**Sketch 8: Profile & Settings**
![Sketch 8: Profile & Settings](../assets/UI%20Sketch-8.jpg)
