# Building a Digital Twin for a logistics firm

My journey building a Digital Twin (DT) for a logistics firm -- lessons learned included.

## Contents

- [What is a Digital Twin (DT)?](#what-is-a-digital-twin-dt)
- [The transportation operations](#the-transportation-operations)
- [MVP](#minimum-viable-product-mvp)
- [Narrowing scope](#narrowing-scope)
- [Designing data models and storage for specific use-cases](#designing-data-models-and-storage-for-specific-use-cases)
- [Making DT scenarios easy to support](#making-dt-scenarios-easy-to-support)
- [Exploring more of the problem space](#exploring-more-of-the-problem-space)
- [Getting data in more hands](#getting-data-in-more-hands)
- [Deploying models behind APIs](#deploying-models-behind-apis)
- [Improving data storage and endpoint performance](#improving-data-storage-and-endpoint-performance)

### What is a Digital Twin (DT)?

I work for a logistics firm called [NFI Industries](https://nfiindustries.com). The company offers a variety of services and resources to other businesses to manage or support their supply chains.

The company has a division referred to as Integrated Logistics. Under that umbrella is a department called Transportation Management (TM). You can think of that department as a kind of business outsource that specializes in transportation routing and related services.

I've found this introduction to the business very important to clarify early on when discussing what a DT is. To be honest, I prefer to think of a DT as data infrastructure and services to support your business' scenario modeling. So it's a system that specializes in serving users data about different business scenarios.

For a TM department a DT encompasses the entire capability of reproducing the execution of a customer's routes.

### The transportation operations

To understand our DT in more detail it's important to get to know the core operations we model.

The TM department is responsible for the delivery of product throughout the customer's supply chain network. This could mean moving product from a plant to an end retailer. 

<p align="center">
  <img src="https://github.com/cnpryer/cnpryer.github.io/blob/main/assets/img/simple_supply_chain.png?raw=true" alt="Simple Supply Chain" width="50%">
</p>

Along the way might be several steps. And each step might vary in complexity. 

<p align="center">
  <img src="https://github.com/cnpryer/cnpryer.github.io/blob/main/assets/img/complex_supply_chain1.png?raw=true" alt="Complex Supply Chain" width="70%">
</p>

Our TM team can comb through your network and prescribe routes based on your specific needs. As a provider we're able to optimize things like transportation costs by implementing clever strategies like backhaul matching based on volume to, from, and across regions.

The TM team leverages another system developed specifically for logistics firms like ours. That system houses all of the integrations, planning parameters, and other constraints for routes to be executed using. The system gives our business the ability to model customer-specific profiles for routing operations.

The routing operations **will** vary by customer. This is an important aspect of NFI's value proposition. So we are modeling the operations surrounding and executed via that specialized system.

The DT is a separate system used to emulate that entire operation.

### Minimum Viable Product (MVP)

I started out at NFI developing specialized models for our customers. The first few years of my career I spent a lot of time on complicated projects that took several months to deliver. The work gave me unique experience building supply chain network and transportation models at scale for large businesses.

This experience allowed me to learn what about these models are often found valuable in an engagement with a customer. It also helped me learn useful methods for experimenting with feedback loops during these projects.

I don't really like to consider an MVP as *one thing*. To me it's a means to an end. I use an *MVP phase* to gauge interest, and I do what's necessary to establish support for areas of interest to consider.

The models I developed earlier in my career often highlighted high-value operational wins for the customer. So the first step of the MVP phase was to gauge interest in some of those operational considerations.

After we established that there was interest in understanding the quantifiable impact of various operational decisions, we wanted to develop a small scale model to give our customers something to interact with.

We served up a basic dashboard with a model's results for a set of low-hanging-fruit scenarios.

### Narrowing scope

This really should be treated as a massive endeavor, but we're cognizant of this and implement an aggressive scope-trimming strategy. Our process is very feedback-oriented. I want to make sure we're working on things that make sense to work on. You can't do that without being able to test enhancements with the customer.

We knew we needed to figure out an early-stage interface for our customers to interact with. Our goal was to make the delivery of new scenario data as straight-forward as possible to support and consume.

We also understood how complicated our problem space could get. Not only are there customer profiles for route management, but depending on how your routes can be executed the solution might change. The *Vehicle Routing Problem* (VRP) is a domain in and of itself. VRP is [NP-Hard](https://en.wikipedia.org/wiki/NP-hardness). 

In practice there are methods for exact solutions, but as the problem grows in size and complexity exact solutions become incredibly more difficult to derive. However, in industry and especially with our business, we're able to break the problem down enough to generate exact solutions. Our bread and butter is the implementation -- being able to execute exact routing for our customers is critical.

When we're unable to break the problem down sufficiently we have tools that help us apply heuristics for less exact solutions.

TM operates all kinds of transportation services. There are many ways to slice routing, so I'll start with one: stop sequencing.

Rules can be applied to your stop sequencing logic. This is necessary at times if customers are moving product that would be best managed a certain way. For example, if you are maximizing your level of service for target retailers, you might implement a strict stop constraint; such as limiting the number of stops on any route to only a couple.

The DT becomes much more difficult to build if we're not careful with the elements we intend to model for early scenarios.

To start, we focused on specific levers (or slices) for our customers:

- Constraints for stop sequencing
- Constraints for ship windows
- Constraints for carriers
- Constraints for aggregation

The key is to focus on high-value parameters our customers can interact with. By identifying parameters and constraints that consistently drive actionable improvements to our customers' supply chains, we're able to approach our customers sooner and focus on potentially implementable solutions.

Deciding what to work on and when to do it is important to me -- and to the team. We want to make sure we're working on things that make sense to work on.

We selected four of our customers to start out with based on a combination of their current operations, scale, and existing relationships.

### Designing data models and storage for specific use-cases

Early on we realized our existing BI infrastructure would work against us. Historically, dashboards have been built off of a monolithic reporting server developed for business analysts and custom portals. It worked for its use-cases, but for the DT we needed to be able to give customers access to a lot of data. And we wanted to make sure to create an experience they'd want to come back to.

The key characteristic of the dashboard we wanted to support early on was support for scenario-to-scenario comparison.

A dashboard user can have many scenarios. We want those scenarios to be comparable.

Beyond just views for your scenario data, what makes this dashboard useful is what data it gives you access to. So your bottleneck becomes how many scenarios you can build, and the ways you can share and interact with that data.

If the scenario is actionable, you want to go from review to validation as fast as possible. That's a hard problem to solve. It's a mesh of trust, tools, people, and process. We often say the DT should be a tool people can trust and use in their processes.

So we wanted to make sure we do a few things really well:

- Build trust fast
- Build flexible tools
- Get this in the hands of more users
- Demonstrate DT in downstream apps

We had two clear use-cases to support early on:

1. A scenario-vs-scenario view of your data
1. An interface for running a scenario

#### A data model for storing source data

Let's focus on the specialized system our TM team leverages to execute routing. More specifically we'll focus on the data used by the system for routing.

There are two key elements: orders and route guides.

You can think of orders as the product. An order can have scheduling. It has a pickup and delivery location. The system is used to route orders according to a route guide -- or the customer's routing profile.

Route guides act as a profile for routing logic. This consists of rates, rankings, allocations, other constraints, etc.

To emulate these operations we need to store this important data in a way where we can reliably produce routing scenarios from.

At a high level we store the following as source data for the DT:

- Line items
- Stops
- Routing profiles

This is an oversimplification of our source data storage. There are many more entities. Being source data, there are also less processed formats available for our models.

The main use-case the source data supports is generating model data.

#### A data model for storing dashboard data

Scenario data isn't helpful without methods for consuming it. Our customers currently access other data through existing BI, so we wanted to add a view for their scenario data. The main use-case for this data is review and comparison of scenario data.

The design of this storage optimizes query performance for the dashboard. We want our customers to have access to as many scenarios as they deem valuable for their business. We also don't want that requirement to produce a slow and frustrating experience.

Typically reporting on routed orders (or line items) is done with a pickup-and-delivery format. So the database maintains tables already transformed for that view.

A lot of computations are done with a pickup-and-delivery format. So this design supports comparison of many KPIs that we already provide visibility for in existing BI.

We want the experience to feel familiar, but at the same time we want to leave room for necessary innovation.

#### A data model for scenario runs

> ⚠️ Disclaimer: This is currently a roadmap item.

A [data model for source data](#a-data-model-for-storing-source-data) and a [data model for dashboard data](#a-data-model-for-storing-dashboard-data) cover most of the first use-case mentioned in [designing a data model for specific use-cases](#designing-data-models-and-storage-for-specific-use-cases) ("1. A scenario-vs-scenario view of your data").

For the second use-case ("2. An interface for running a scenario") we needed to be careful with how we produce data for existing and new scenarios. Naturally your baseline is a kind of scenario. A new scenario can be developed from your baseline scenario. You can think of it like a scenario branch.

If you're building a scenario that adds a new carrier to the customer's route guide you'll need more source data. If you're building a scenario that adds that new carrier **and** modifies a constraint, you're going to want to avoid having to introduce the new route guide data again.

<p align="center">
  <img src="https://github.com/cnpryer/cnpryer.github.io/blob/main/assets/img/simple_scenario_branch.png?raw=true" alt="Simple Scenario Branch" width="50%">
</p>

One way to do this would be to store model transformations with versioning. So scenario runs can reference data from already implemented transformations.

If you're building a scenario off of another, you might need to *search* for that scenario before you create your new scenario. Fetching a previously run scenario can be done with indexed scenario names, labels, modifications, and more.

### Making DT scenarios easy to support

If you have never supported a scenario where a user can add a carrier to a route guide then there is upfront work to offer that.

We've always understood the value of branching our scenarios, but we are approaching a long-term solution in manageable steps:

1. First give the developer the ability to implement ingest and transformation for the scenario
1. Tighten that delivery cycle as much as possible with current paradigm (we are here)
1. Abstract logic behind APIs
1. Test distributable infrastructure with PoC clients
1. Iterate on infrastructure based on use-cases

The more we can re-use for new scenarios the easier it is to build them. The easier it is to build scenarios the more distributable that process is. The more distributable the scenario-building process is the more scenario data our customers have access to. The more scenario data there is in the hands of our users the more potentially implementable decisions can be made. 

If there are more implementable decisions it becomes much more likely for action to be taken directly from a DT-embedded workflow.

### Exploring more of the problem space

One of the lessons we learned early on was how truly valuable feedback was for development. Before we built up intuition towards a scenario-vs-scenario requirement, we learned how users interact with dashboards already part of TM's BI.

There is both a technical and a domain side to the problem we are solving.

The technical side is all about understanding what is required to build a system like this. What systems should we leverage and what systems create challenges for the DT.

There's a lot we learned about users of existing TM BI. To keep this simple I'll focus on two: granularity and searchability.

Our users wanted as much grain as you can provide. This is crucial to establish trust and support existing processes. Our users already digest route data. It's just their current baselines. So there is historical data we track KPIs and all kinds of information for already. That data is exact and at its actual grain.

So if a user wanted to understand a route order-by-order, and down to the class of the product, they would pull that data and do their own analysis on it.

Users relied heavily on *searching* for information. A lot of decisions are made at the node-level in a supply chain network. For example, to support a reliable level of service certain carriers might be preferred. Users would perform analysis on locations using data like their city, the facility name, and a carrier name or SCAC code.

We knew that whatever we built would need to offer searchable methods for reviewing granular route detail.

More importantly there is the domain side of the problem.

We intentionally limited scope so that focus was on hot paths for value. If modifications to specific constraints yield consistently interesting insights then we want to prioritize those. This is great for moving quickly, but there is a cost. You're biasing your exposure to your problem space.

It's possible there are unique scenarios that provide as much (if not more) value as the low-hanging-fruit we've focused on. In fact, we actually have experience with this already.

We've developed scenarios that specifically modify constraints at a location-level where a new mode was introduced to their network. In our experience this kind of mode conversion is difficult to achieve. Specifically it was introducing intermodal to their network -- which requires coordination from as far upstream as the vendor to as downstream as the individual retailer.

This is often very difficult to implement since vendors and retailers possess some of the most influence in the operation.

<p align="center">
  <img src="https://github.com/cnpryer/cnpryer.github.io/blob/main/assets/img/intermodal_scenario.png?raw=true" alt="Intermodal Scenario" width="70%">
</p>

We knew we had to open up the flood gates just a little to expose ourselves to the unique scenarios our customers might want to develop.

### Getting data in more hands

As the DT becomes more available across the business more users will interact with, create, and share their scenarios. Decisions in a logistics firm are often made based on data and coordination between various operations experts. These are individuals at NFI who have years of experience working directly with their customer's routing.

To roll out the DT to more users, we've crafted roughly the following roadmap objectives:

1. Run scenarios across models and dump that in a data mart
1. Promote DT dashboard development with analytics group
1. Introduce users to experimental methods for sharing and analyzing DT data
1. Push actions up the NFI leadership chain
1. Distribute infrastructure for user-owned scenarios
1. Promote downstream development for DT-embedded applications

After finding success driving actionable decisions with our customers, we've introduced more consumers of DT data. We believe that if we continue to develop the DT with our customers we'll eventually need to match the scale in demand for those scenarios. A scale where the DT is embedded in the user's workflow.

### Deploying models behind APIs

> ⚠️ Disclaimer: This is currently a roadmap item.

The DT is responsible for several massive changes to our customers' operations. These are huge wins for the project, and for our customers, but there are lessons we've learned that we have not yet addressed.

The main piece of both explicit and implied feedback we've consistently received working with our customers is the ability to build and run a scenario of their own.

There is a lot to unpack in that sentence.

First, *"their own"* scenario means there is a concept of ownership of scenario data. This idea is supported by explicit feedback from our customers where they often expect to have the ability to take their route guide, modify it, and then run that as a scenario. This idea is also supported by implicit feedback we've observed.

We give scenarios names in our data. So there is a string that is used as the name of a scenario we designed with our customers. The name of scenarios is used to both identify the scenario as well as store information about the scenario.

One user might call a scenario something another user may understand better in another way. This is actually a big deal since our ability to drive actionable decisions is limited by the user's ability to share that scenario's story.

The scenario is a concept the user develops, so it should be stored with that data. When scenario data is reviewed and shared the name is not as relevant as the story being introduced. The user may want to share that story differently depending on the scenario.

Besides semantics and how, in this case, it influences UX and our ability to create value for our customers, the fundamental capability of building and running scenarios is a necessary problem to solve at a greater scale.

You might throw a few data engineers at this to build out the storage and pipelines for these models, but you will only ever support the development of scenarios your data engineers can deliver. Throwing more data engineers at the problem is not going to satisfy the scale our users expect scenarios to be delivered at.

At the moment we believe the best way to solve this problem is to develop these scenarios with the intention to distribute them behind APIs that allow other users and systems to run and modify them.

### Improving data storage and endpoint performance

*Revisiting the backlog.*

Understanding this is a user-driven workflow helps us make sense of what a future DT could look like. We can develop scenarios for our customers, but we only dump runs into a dashboard backend. Each customer has access to what is dumped into that database.

The next step currently planned for the project is delivering those models in a way where their runs and input parameters are available on-demand.

This means we want to support endpoints that can be used from a client. We want the client to give the user the ability to run the scenario they would like to review with their team.

There are many things we've learned about the data storage side of this problem.

One is that you want to support front-loading the transformation of model data as much as you can. This means that if you're running a solver, the format of target input data can be slow to produce given our source data structure.

Going from historically pickup-and-delivery representation to a partitioned sparse matrix indexed by stop pairs can be resource intensive.

In addition, we also want to avoid running redundant compute. And keep in mind, these models float around an NP-Hard problem space (VRP). Without due-diligence models built incorrectly might take too long to run. Or the experience needs to be developed around solve times.

Improving our data storage for this vision and supporting efficient endpoints is a very interesting topic that our team is focused on for 2025.
