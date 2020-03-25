# Design Document Guidelines

## What is a design document
A design document is a document that describes a technical problem and provides a solution on how to solve it. A good design document must be written for S/M/L/XL projects. A good design document is described [here](https://w.amazon.com/bin/view/AWS21/EngineeringProcess/HowToWriteADesignDocument/#HWhatisagooddesigndocument3F). A design document should not have more than six pages of Word documents.

## How to write

### State your goals
State the goals of your document. Why did you write the document and what are the key questions that you are trying to answer.
### Remember your audience
Know the targeted audience of your document. Remember to write your document in a way that your audience understands the right level of details. Some terms might not be familiar to your targeted audience and you should be able to define it. You can add a Glossary section for terms that need definitions.
### Focus on the scope
You need to be clear on what the document will cover and what will not. Your narrative should focus on the scope.
### Have a structure
Structure your document in sections. Each section describes an element that is presented clearly and separately.
Aside from sentences and paragraphs, there are several ways you can present your narrative.
1. List
   * Some information are hard to organize and a list is a good way to organize your ideas.
   * You can use bulleted list for unordered items and numbered list for ordered items.
2. Table
   * Another way to present your information is through tables.
   * Tables make it is easy to compare and analyze information.
3. Diagram
   * Diagrams are used to visualize the characteristics of your design.
   * You can use [draw.io](drawio.corp.amazon.com/) to create design diagrams.
   * UML and other tools can be used as well.
Keep in mind to only use list, table and diagrams when applicable.
### Use data
Make sure your assertions and claims are backed-up by data. Did you use data to make informed decisions in the document? If there are gaps in your data but it is still worth stating in the document, address those gaps and point out the holes.
### Engage the right persons
Reach out to persons who have feedbacks and insights to make your document stronger. In the middle of the writing process, have a review with someone on the team who has the expertise or knowledge that can provide insights on what you have already written on the document. This avoids surprises on major design choices that might not be feasible or something that doesn't conform with the team's standards.
### Signed-off by the team
The document should be reviewed by the team. This is to ensure that major design decisions has been agreed by the team and gaps in the documents will be filled based on feedbacks from the review.

## What to write

## Must have
Depending on the type of the project, the content of the design document will vary and here are the sections that every design document must have.

#### Overview
This provides a brief description of the document.
#### Scope
This defines the boundaries of the project. It is important to pin down the scope as it affects the cost and timeline of the development of the project. If there are related topics that are out of scope and worth mentioning in the document, create an out of scope section for that.
#### Goals
Provide the business and operational goals of the project.
#### Requirements
The requirements should be defined in this section and can be classified as:

##### Functional Requirements
These are the features that must be implemented to complete the project. If possible, tag and list down the requirements in order of priority.

##### Non-functional Requirements
Non-functional requirements define the quality of a system such as security, reliability, performance, and scalability. Here are some examples:
1. Metrics
2. Tests
3. Alarms
4. Dashboards
6. Weblab
5. SLA(Latency, Error rate)
7. ANVIL classification

##### Technical Requirements
These are the technical tasks that you need to be accomplished for the project to succeed. Infrastructure setup, pipelines, web frameworks are some example of the technical requirements.

#### Design
Describe your design in details.
- Provide an explanation on how each component will be implemented or updated. Deep dive and provide as much detail as possible for each component.
- If applicable, provide a diagram on how the components in the design interact with each other.
- If your design is integrated with other systems, provide the details of the integration.
- When designing a migration from an old system to a new system, provide an overview or diagram of the old system and compare it with the proposed design.
- Provide an API design if you are implementing a new API. API design should describe the data structure of the input and output. If possible, provide a sample request and response.
- If your system is creating new tables or database, provide a detailed description of the database design or schema.

#### Cost
Cost(IMR) is one of the major factors that we should consider when designing a solution to a project. Provide a breakdown on the operating cost of each component.

#### Dependencies
Projects will have dependencies and it should be stated in the document. This section provides a clear understanding of the dependencies and what are the things needed inorder to use the dependencies. Dependencies can be upstream services, packages or libraries.

#### Appendix
There are information that can be too detailed to be added in the document as it may distract the reader. These information can be included in the appendix. FAQs can also be added in this section.

## Recommended Sections

### Milestones
Milestones are useful to mark the stage in which the project is currently at. It shows the progress of the project and communicate what's going on with the project.

### Risks
Every project has risks, it can be low impact or high impact. It is important to identify those risks and how it will affect the project.

### Failure mode
This describes on how your propose system will fail. You should be identify these scenarios and provide mitigation actions. For example, what happens when one of the dependencies is unavailable? Should there be a retry call to the dependency? What's the retry strategy?

## Optional Sections

#### Glossary
This is a collection of terms that was introduced or used in the document. Either define the new terms or link the existing terms to it's definitions.

#### UI Mockup
Some projects require implementing features based on a new or updated user interface design. UI mockups can be added to the document to provide a visual representation of the functional requirements.

#### Alternative options
There can be multiple design proposals for a project and it's important to explore several solutions to weigh the pros and cons and to provide an argument on why you decided on the proposed solution.

## Where to write
Write your draft document in an offline platform. The document will undergo a review and multiple revisions. You can choose where to write your document and below are the suggested options:

1. [Quip](https://quip-amazon.com) - this is good for writing collaborated documents. Quip supports inline commenting which is good for tracking feedbacks.
2. [Workdocs](https://amazon.awsapps.com/workdocs/index.html) - web-based tool to create and edit documents.

Wiki should be avoided when writing the initial document and only when the document is reviewed and finalized then it can be published to the team's [Tech Design Wiki]().

## References
- [Narrative Best Practices](https://w.amazon.com/index.php/Narrative_Best_Practices)
- [How to write a good design document](https://w.amazon.com/bin/view/AWS21/EngineeringProcess/HowToWriteADesignDocument/)
- [Wikipedia: Software Design](https://en.wikipedia.org/wiki/Software_design)

