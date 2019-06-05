# Sealed Rust - The Pitch

This is the first post in a series detailing Ferrous System's plan to qualify the Rust Language and Compiler for use in the Safety Critical domain. We call this effort **Sealed Rust**.

Although Rust as a language has been stable since 2015, use of a programming language and compiler within projects that have safety critical demands, such as Automotive, Industrial, or Avionics, requires a higher bar of entry than the Rust compiler can offer today. This typically includes "certification" of a compiler, which is difficult, if not impossible to do with today's versions of Rust as-is. However, we see the path towards using Rust in these domains as entirely possible within the next few years. This process would add Rust to the list of languages offered to developers in these domains, which is currently primarily served by languages such as C, Ada, and C++.

As a modern software development project, the Rust Programming Language and the Rust Compiler as a project has shown to be a best-in-class example of design and development as an Open Source Project. These practices include items such as a formal design process through RFCs, mandatory code review on changes through GitHub PRs, and extensive automated testing through the use of Continuous Integration. These practices are typically hard requirements for safety critical components, and positions the Rust language and compiler in an excellent place to approach formalization and certification in many domains.

## Beginning the Process

We believe that this is a process that will take a significant amount of time and effort to see realized, but something we choose to do for the long term value we believe this will provide to both the Rust Programming Language, as well as industries developing safety critical software. We also see this process not only as a theoretical possibility, but a task that can be realized in the scale of the next few years.

We see this effort as a bit of a "moonshot". There is a lot of work to be done, however, there is a tremendous amount of value here. There is significant interest in the use of Rust in safety critical domains, and We share [Carol Nichols]' characterization of Rust as being [a language for the next 40 years].

[Carol Nichols]: http://twitter.com/carols10cents
[a language for the next 40 years]: https://www.youtube.com/watch?v=A3AdN7U24iU

We certainly could wait for the Rust project to organically make it closer to the goal of being usable in these domains, however we are interested in making that happen sooner than later. Because of this, Ferrous Systems is looking to organize these efforts, despite it being a daunting, but not an insurmountable, challenge.

If you are coming from a Rust background, consider this series of blog posts and the associated plan in the context of a Pre-RFC: A rough sketch not intended to be perfect, but to lay down the facts and challenges as they are seen today by us, and to focus the discussion to find the details necessary to address before a decision can be made whether to start down this path. We hope to gather feedback, as well as parties interested in seeing this process become successful.

We will first be releasing this plan in parts in blog post format, and we will be capturing the details of each post in our [coordination repository]. The outline of posts in this series is:

* [The Pitch (this post)](.)
* Background - The Rust Language and Safety Critical Development
* Preparing for Sealed Rust
* Specifying the Rust Language
* Verifying Against the Specification
* Paperwork - IEC 61508, ISO 26262, DO-178C, and friends
* The First Certification
* Steady State and Expansion

[coordination repository]: https://github.com/ferrous-systems/sealed-rust

There is a lot of material that needs to be covered here, but rather than starting with details, we will instead start with what this could look like. The details here are all subject to change.

## A Peek at our Vision

Currently, the Rust Language has three states of stability: Nightly, Beta, and Stable. Rust has also introduced a cycle for making breaking changes to the language, referred to as Editions, which are based on the year that they were released, e.g. 2015 and 2018.

We would propose a fourth level of stability, aimed at usage within safety critical domains called **Sealed Rust**. This version of Rust would be a subset of the greater language, initially containing a subset of the features provided by `#[no_std]`, and a subset of the `core` library. The contents of Sealed Rust would contain parts that have been formally specified by and verified against **The Rust Specification**, described below.

**Sealed Rust** would be released on a similar cadence as the Rust Editions, offset by 18 months or so. This time delay would allow time for specification, validation, and time to learn lessons from early releases in the currently stable edition. Sealed Rust would not follow the stable 6 week release cadence, and would instead only be revised if security or correctness fixes are necessary, releasing only one major version per edition. When using **Sealed Rust**, only compiler features or library components that have been sealed may be used.

Components of the compiler and `core` or `std` libraries would need to go through a stabilization process called **sealing** before being included in **Sealed Rust**. Similar to the current process required to move from `nightly` to `beta`, or `beta` to `stable`, this process is subject to review before a component could be considered `sealed`. Additionally before **sealing** can be completed, the compiler or library component must have an accepted specification in **The Rust Specification**, and must be verified against that specification. Due to the higher effort and formalism required to seal a component, initial effort would focus on parts of the language that have not undergone or are not expected to undergo significant changes. Sealed Rust would offer the same stability guarantees given to Editions in Rust, with one cycle of deprecation required before breaking changes can be made.

In order to support **Sealed Rust**, it will be necessary to develop a specification document for Rust, which we refer to as **The Rust Specification**. This document would contain one section for the compiler itself, specifying the semantics of the Rust language. It would also contain additional section(s) for components such as the `core` and `std` library, specifying the behavior of these components. This specification will be in the style of other language specification documents, such as ISO/IEC 9899:2018 for the C programming language, or ISO/IEC 14882:2017 for the C++ programming language.

This document will be maintained as an open source work, similar to other documentation components, though may be officially published as a standards document elsewhere. The validation tests demonstrating conformance to the specification would also be maintained in Open Source as a new collection of tests.

For many safety critical applications, it is not possible to "certify a compiler" as a blanket process, as it is necessary to take into account the particular use cases and demands of each project. However, we aim to upstream as many of the documents, tests, and reports necessary to perform the final certification back to the Rust project itself, in order to make Rust the easiest language to use for safety critical projects. This would allow certification for final usage by end companies, or by third party service providers with domain expertise.

## Who is this effort aimed towards?

We hope that the **Sealed Rust** effort would be applicable to any developers working in safety critical software domains, such as:

* Automotive (under safety standard ISO26262)
* Industrial (under safety standard IEC61508)
* Robotics (under a number of safety standards deriving from IEC61508)
* Medical Devices (under safety standard IEC62304)
* Avionics (under safety standard DO-178)

While these domains have different constraints and regulatory requirements, we hope to find a plan that will eventually allow use of Rust across all of these domains. This may include taking an iterative approach, targeting projects with lower safety-critical demands, and increasing the scope to higher levels of demands with future efforts).

## Impact to the Rust Project

It is not clear today whether **Sealed Rust** would become part of the Rust Project as a whole, or as a separate project. We plan to discuss all options with members of the Rust Project to determine what makes the most sense, both to reduce dual-maintenance for safety critical use cases, as well as reduce load on the maintainers of the Rust project. Although this project is starting separately from the Rust Project itself, we hope to find a path to combining these efforts in a reasonable way.

Stated briefly, we aim to uphold the following items:

1. We hope to keep the impact to the Rust Language itself minimal for most users
2. We hope to keep the impact to the maintainers and developers of the Rust Language and Compiler minimal

Regarding point 1, by applying the formalism required for the safety critical domain to only a subset of the language, we hope not to affect the velocity or development cycle of either the Rust Language or the Rust Compiler. The hope here is to formalize core components of the language, improving the reliability of these components, rather than "forking" the project for safety critical users. Ideally, this would lead to improved reliability for all users - items that have been formalized will be specified and tested in depth, acting as a strong foundation for concepts built on top of them.

Regarding point 2, we do expect to require feedback and assistance from teams within the Rust project, particularly with regards to what parts of the language to formalize, and the most practical methods to achieve this. However, we hope that the team(s) working on the Sealed Rust effort "give back" more than they "take", acting as a net-positive for the Rust project in general.

## Funding

Although this will be addressed in a longer format later, it is worth mentioning our thoughts around funding earlier than later.

We expect this to be a significant undertaking (likely many full time developer-years of effort), and as such, it is something that will require paid effort to make real.

How we fund this is a bit of an open question, however there are a few possible routes here:

1. We are able to take funding from companies and public funding grants, and are able to deliver the items described above as a fully open source contributions to the Rust project.
2. We are able to secure funding through investments and loans, and are able to deliver some of the items listed above as open source contributions, while keeping some of the items above as proprietary, in order to offer them as a product to interested companies looking to use Rust in a certified context to recoup our costs.

As a company, Ferrous Systems is strongly committed to open source, and hopes that option 1 described above is possible to achieve, as it is our preferred outcome. We see this as a net-good to the safety critical software development industry - lowering the cost and effort required from end companies for developing certified software for safety critical applications.

In either of the options above, we expect Ferrous Systems to perform project management responsibilities, and to act as the coordinator for this effort. We would hope to collect funding openly through a system such as [OpenCollective](https://opencollective.com/), and distribute the funding received either by expanding the team at Ferrous Systems, or by contracting parts of the work to members of the Rust community, and subject matter experts in relevant domains.

In the long term, we expect that Ferrous Systems, and likely other entities, would offer services around the certified version of the Rust Language and Compiler, including:

* Expanding Sealed Rust to new safety critical domains
* Offering tools and resources for companies using Rust in a safety critical domain
* Providing Long Term Support for versions of Sealed Rust, extending beyond the "base" stability guarantees offered by the Edition Releases

## Where to go from here?

[Ferrous Systems](https://ferrous-systems.com) is a Berlin, Germany based consulting company with numerous ties to the Rust Project both as members of the project, as well as active users of Rust. Based on our experience in the Rust Language and Safety Critical domains, we hope to facilitate the process of moving Rust into this new area. Until the end of the summer, Ferrous Systems will continue to finalize the plan to realize **Sealed Rust**. During this time, we hope to start conversations with stakeholders, including:

* The Core Rust team, and the Rust project as a whole
* Companies interested in using Sealed Rust, such as Automotive component vendors and firmware development companies.
* Companies with domain expertise specifying, validating, and certifying programming languages and tools
* Silicon vendors, interested in providing a Rust environment for customers using their hardware components
* Research universities interested in contributing to these efforts
* Certification Authorities, such as TÜV, the FAA, and EASA
* Companies interested in providing material support, such as engineering assistance and financial support

Once the plan has been finalized, we can begin drawing a road map based on the tasks enumerated, and the level of financial, research, and engineering support available for these efforts. We hope to bring stakeholders together in a form such as a Working Group (WG) or Special Interest Group (SIG), made up of the stakeholders listed above. If you or your company are interested in joining this effort, please [join our newsletter](#), follow our [coordination repository], or [contact us directly] to discuss.

At this early stage, funding is particularly necessary to begin this process. If you or your company are interested in assisting with financing this initial work, please [contact us directly].

[contact us directly]: mailto:sealed-rust@ferrous-systems.com

We can't wait to make this a reality, with all of your help.

> *James Munns*
> *Managing Director*
> *Ferrous Systems GmbH*
