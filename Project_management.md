# Documentation

## Task division

1. `Team Leaders 🠚`Define inputs, outputs and purpose of the issue.
	- An example of use would make the work more understandable.
2. `Team leader 🠚`Assign PR:
	- Developer (developer): Core development.
	- Co-developer (tester): In charge of the tests and the continuous testing.
3. `Assigned pair 🠚`Analyze issue and narrow scope.
4. `Assigned pair 🠚`Propose changes on the issue description and ask for more information if necessary.
4. `Assigned team 🠚`Estimating times to get to the code review.
5. `Assigned pair 🠚`The tester keeps track of the PR. 				
6. `Assigned team 🠚`When the tester and developer reach a consensus, the rest of the team performs the review.

## Terminology
The following terminology is [google's internal technology](https://google.github.io/eng-practices/):
- **CL** (Change list): Is one self-contained change that has been submitted to version control or which is undergoing code review. Sometimes called "change", "patch" or "pull-request".
- **LGTM** (Looks Good To Me"): It is what a code review says when approving the a **CL**.

### Code review
A key point here is that there is no such thing as *perfect code*, there is only *better code*:
- The review should balance between the need to make forward progress and the importance of the changes suggested.
- A CL that improves maintainability, readability and understandability of the system shouldn't be delayed because isn't perfect.

#### Review labels
This labels are a consensus. They can be linked to a specific section of the code or one of the explicit review sections (design, functionality, etc.):
- Severe: Merge blocking. Could lead to errors, performance issues or has problems with the first three review section.
- Improve: Merge blocking. Minor change, better to address before the merge.
- Nitpick: Minor change, not an error but an observation.
- Question: Inquire about something, could lead to one of the previous labels.
- +1: Congratulate a teammate about his way of doing things!

### Code reviewer perspective
Is a process in where someone other than the author(s) of a piece of code examines that code.

#### What you should be looking?
The following list is in descending order of interest. One should 
- **Design**: 
This is the most important thing to cover in reviews, the overall design of the CL.
	- Do the interactions of various pieces of code in the CL make sense? 
	- Does this change belong in your codebase, or in a library? 
	- Does it integrate well with the rest of your system?
	- Is now a good time to add this functionality?
- **Functionality**: 
	- Does this CL do what the developer intended?
	- Is what the developer intended good for the users of this code? (users could mean end-user or future developers).
	- Is the way code behaves good for its users?
- **Complexity**: 
If code is too complex it usually means that "can't be understood quickly by code readers" or that "developers are likely to introduce bugs when they try to call or modify this code".
	- Could be made simpler?
	- Would another developer be able to easily understand and use this code in the future?
- **Tests**:
	- Does the code have correct and well-designed automated tests?
- **Naming**:
	- Did the developer choose clear names for variables, classes, methods, etc?
- **Comments**:
	- Are the comments clear and useful?
- **Style**:
	- Does it follow the coding style?
- **Documentation**:
	- Did the developer also updated the relevant documentation?

