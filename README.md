# Approval Process
This library gives you the ability to implement a generalized approval process in your project where multi-level approval process is necessary, for example, you have a Human Resource Management System (HRMS) or a Customer Relationship Manangement System (CRMS) where approval of certain activity is found very often. 

This library is implemented using java 1.8 and will support any java version from 1.8 onwards. You can also extend its functionality and make it serve your needs within its boundary. Just a disclaimer before diving into the how to guide, this library is not available on maven central so follow the steps to add this into your current project.

# How to guide

1. clone the project on your local machine with git clone command
2. navigate to the project directory
3. inside the libs directory you will find the approval.jar
4. copy the jar file
5. create a folder location inside your project's resource directory, for example, /src/resources/libs/
6. paste the approval.jar file inside the libs directory
7. it is assumed that your are using a build tool for your project i.e. maven or gradle
8. if so, then add the dependency path inside your pom.xml for maven, or build.gradle file for a gradle project
9. update the project
10. you are good to go!

# Add as a maven dependency

```maven
<dependency>
	<groupId>approval.process</groupId>
	<artifactId>approval-process</artifactId>
	<version>1.0.0</version>
	<scope>system</scope>
	<systemPath>${project.basedir}/src/resources/libs/approval-process-1.0.0.jar</systemPath>
</dependency>
```
# Example code

```java
import java.util.Objects;
import java.util.Optional;

import approval.process.ApprovalWrapper;
import approval.process.RequestLifeCycle;
import approval.process.Reviewer;
import approval.process.Step;

public class Test {

	private static class ReviewerModel {
		private long id;
		private String name;

		public ReviewerModel(long id, String name) {
			super();
			this.id = id;
			this.name = name;
		}

		@Override
		public int hashCode() {
			return Objects.hash(id, name);
		}

		@Override
		public boolean equals(Object obj) {
			if (this == obj)
				return true;
			if (obj == null)
				return false;
			if (getClass() != obj.getClass())
				return false;
			ReviewerModel other = (ReviewerModel) obj;
			return id == other.id;
		}
	}

	public static void main(String[] args) {
		ApprovalWrapper approvalWrapper = new ApprovalWrapper();
		ReviewerModel rm1 = new ReviewerModel(1, "R1");
		ReviewerModel rm2 = new ReviewerModel(2, "R2");
		ReviewerModel rm3 = new ReviewerModel(3, "R3");
		Reviewer<ReviewerModel> r1 = approvalWrapper.addReviewer(rm1);
		Reviewer<ReviewerModel> r2 = approvalWrapper.addReviewer(rm2);
		Reviewer<ReviewerModel> r3 = approvalWrapper.addReviewer(rm3);

		approvalWrapper.createApprovalStep(r1);
		approvalWrapper.createApprovalStep(r2);
		approvalWrapper.createApprovalStep(r3);
		RequestLifeCycle requestLifeCycle = approvalWrapper.requestProcessLifeCycle();
		System.out.println("request life cycle: " + requestLifeCycle);

		Optional<Step<?, ?>> optionalStep = requestLifeCycle.getValidatedCurrentStep(r1);
		if (optionalStep.isPresent()) {
			Step<?, ?> currentStep = optionalStep.get();
			currentStep.accept();
		}
		// approvalWrapper.removeStep(approvalWrapper.addReviewer(1L),
		// requestLifeCycle);
		System.out.println("after approval by 1L: " + requestLifeCycle);
		System.out.println("is completed?: " + requestLifeCycle.isComplete());

		Optional<Step<?, ?>> optionalStep2 = requestLifeCycle.getValidatedCurrentStep(r2);
		if (optionalStep2.isPresent()) {
			Step<?, ?> currentStep = optionalStep2.get();
			currentStep.accept();
		}
		System.out.println("after approval by 2L: " + requestLifeCycle);
		System.out.println("is completed?: " + requestLifeCycle.isComplete());

		Optional<Step<?, ?>> optionalStep3 = requestLifeCycle.getValidatedCurrentStep(r3);
		if (optionalStep3.isPresent()) {
			Step<?, ?> currentStep = optionalStep3.get();
			currentStep.accept();
		}

		System.out.println("after approval by 3L: " + requestLifeCycle);
		System.out.println("is completed?: " + requestLifeCycle.isComplete());
	}
}
```
