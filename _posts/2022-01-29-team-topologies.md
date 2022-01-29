Book Review: Team Topologies

[Team Topologies](https://www.goodreads.com/book/show/44135420-team-topologies?ac=1&from_search=true&qid=hFfpZOUpUu&rank=1) is an excellent read for engineering managers if you are curious about how to organize teams and their interactions. Many engineering managers have experienced the pain of hand-offs between teams going awry, lack of clarity in team interactions causing frustration, and too many integration meetings that drain energy in your day. Team Topologies is a read to help your teams reduce those pains and have more fun delivering software.

First, the authors provide an excellent vocabulary to discuss how teams organize and interact. They provide definitions for four types of teams:
* Stream-aligned Team - continuous stream of work aligned to the business
* Enabling Team - a specialist that increases the stream aligned teams capabilities through collaboration
* Complicated Sub System Team - Build and maintain a part of a system that requires specialized knowledge
* Platform Team - Enable stream aligned teams to deliver with substantial autonomy

And three interaction types to use amongst teams:
* Collaboration - working closely together with another team
* As a Service - consuming or providing something with minimal collaboration
* Facilitating - helping (or being helped by) another team to clear impediments

In addition to terminology, there are numerous concepts in the book that resonated, and three ideas that resonated most were to:
1. Organize teams for flow
2. Limit Cognitive Load
3. Restrict communicaion

Let's dive deeper into those three.

## Organize for flow

Organizing for flow has Conway's Law at the foundation. Conway's Law states that software teams are restricted in their architecture based on their communication flows. To build a fast-flowing team, consider using the Reverse Conway Maneuver, as coined by the authors, and think about the teams first, and the team structures will drive architecture. Those team structures can focus on limiting hand-offs similar to the [Spotify video](https://www.youtube.com/watch?v=Yvfz4HGtoPc). In practice, cross-functional teams with multiple skill sets concentrate on solving a well-defined problem, resulting in teams with fast flow.

_"The goal is for your architecture to support the ability of teams to get their work done."_

The natural tendency in software is to create teams based on skills, not on the problem or problem domain delivered to the end-user. Front End Developer, Back End Developer, QA, and DBAs teams would be an example of organizing by skill. These team organizations can work and seem to work until they don't. You'll run into integration issues, misaligned priorities, or collaboration struggles when it doesn't work. The skills-based teams approach results in hand-offs for a unit of work to be completed vs. delivering end-to-end value. Hand-offs create scheduling/prioritization and timing issues and slow down delivery. When several skill-based teams provide features for a product, the planning of features by product and project management can face immense challenges. Teams with clear focus and boundaries to deliver software minimize scheduling, management, and prioritization. 

The authors would propose that you can and should create the teams and organize around the work you need to be delivered. Managing a team around a value stream also increases focus autonomy and distributes decision-making. In team types nomenclature, create a stream-aligned team to deliver 80% of the work. Enabling teams facilitate specialties of the stream-aligned team in support. The enabling teams may contribute members directly to the stream-aligned team or operate in a facilitating mode. If you create in this manner, the architecture will follow the pattern of your team structures.

In practice, requesting full-time resources with different specialties to a project team can result in pushback. Acquiring all software specializations for a software project can not keep the two-pizza rule intact. Budgets also matter. If a specialization team is tight-knit, it can be tough to make a change to have a specialization member on a team full time. People are working for the people next to them, and management disruption of the team should be well understood before making that change.

## Limit Cognitive Load

What do you do when a successful and efficient team appears to be slowing down? Successful teams slowing could indicate that the team has acquired too many domains, too much responsibility, and is losing focus due to their success. To exasperate the issue, ample evidence that the technical knowledge to be a software engineer is increasing, not decreasing. The foundation developers are using and the systems they deliver are growing in complexity. The result is that organizational designers have the challenge of managing cognitive load for teams.

The authors suggest finding fracture planes; seams in the code to break down the problem. Perhaps the fracture plan would become a platform service interface. Teams can look for fracture planes in software architecture to divide responsibilities into cohesive groupings. The authors suggest focusing on dividing teams and their duties using the Reverse Conway Maneuver. The new team structure would naturally alter the architecture to fit the new communication paths. The trick is to anticipate that architectural changes will occur. Limiting cognitive helps teams focus and remain tight-knit vs. spreading thin across multiple disparate domains and requires change management.

## Restrict communication

Restricting communication is a bit counterintuitive, aren't we supposed to over-communicate? The catch here is that a high amount of contact with many people takes substantial time and energy. We can achieve a faster flow if we overcommunicate in our local team and limit communicating with adjacent teams. Collaboration, generally, adds additional time for software delivery. A well-documented service interface can help minimize inter-team communication. Multiple review cycles with multiple reviewers also slows progress. 

There are indicators for over-communicating. Huge stand-up meetings, large group chats, lengthy review cycles, and prolonged interface discussions may indicate room for improvement to limit intra-team communication.

All teams must have a clear purpose and responsibilities. To limit over communication, leaders can alter the problem domain, existing team, and the physical and virtual space. Leadership can create new to divide software domain responsibilities or provide platform services. Teams that are enabled to make decisions have well-defined APIs available and have the right mix of enabling teams using the suitable interaction modes operate in an environment to deliver software faster. 

## Summary

Team Topologies provides a toolbox to apply to your organizational design needs. The vocabulary and interaction modes can help you discuss how you organize, and the team first approach is compelling. The common language provided, diagrams and examples can enable productive conversations for the organization to make progress continually. The concepts take time to apply in practice due to changing project needs, organizational needs, and how software projects are architected and delivered. 

Team Topologies will be a yearly read for me each year. I suspect a different concept will stand out, and I'm two for two thus far.

On paper, creating stream aligned looks straightforward. In practice, you're working with aligning unique people and team interests, which is always challenging. Achieving fast flow on a team is an ongoing effort. The needs of a project change, the needs of the people on the project change, and the organization's needs are changing over time. Change is all around us, and keeping your teams productive requires consistent effort.