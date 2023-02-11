# Request Approval Process
This library gives you the ability to implement a generalized request approval process in your project where multi-level approval process is necessary, for example, you have a Human Resource Management System (HRMS) or a Customer Relationship Manangement System (CRMS) where approval of certain activity is found very often. 

This library is implemented using java 1.8 and will support any java version from 1.8 onwards. You can also extend its functionality and make it serve your needs within its boundary. Just a disclaimer before diving into the how to guide, this library is not available on maven central so follow the steps to add this into your current project.

# Signature verification

This library is digitally signed. In order to verify its authenticity openup a terminal within the directory and run the following command

```jarsigner -verify approval-process-1.0.0.jar```

You will find the output similar to below if it the library is verified, please note that certificate was not issued by any trusted Certificate authority.

```
jar verified.

....
```

# How to guide

1. clone the project on your local machine with git clone command
2. navigate to the project directory
3. inside the project directory you will find the approval-process-1.0.0.jar
4. copy the jar file
5. create a folder location inside your project's resource directory, for example, /src/resources/libs/
6. paste the jar file inside your created directory
7. it is assumed that your are using a build tool for your project i.e. maven or gradle
8. if so, then add the dependency path inside your pom.xml for maven, or build.gradle file for a gradle project. In case you are not using any build tool just add the jar file as an external dependency to your project
9. update the project
10. you are good to go!

# Add as a maven dependency

```maven
<dependency>
	<groupId>approval.process</groupId>
	<artifactId>approval-process</artifactId>
	<version>1.0.0</version>
	<scope>system</scope>
	<systemPath>${project.basedir}/src/main/resources/libs/approval-process-1.0.0.jar</systemPath>
</dependency>
```

# Add as a gradle dependency

```gradle
dependencies {
    implementation(files("/src/main/resources/libs/approval-process-1.0.0.jar"))
}
```


# Example code

```java
package approval.process.test;

import java.util.HashSet;
import java.util.Objects;
import java.util.Optional;
import java.util.Set;

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

		@Override
		public String toString() {
			return "ReviewerModel [id=" + id + ", name=" + name + "]";
		}

	}

	@SuppressWarnings({ "unchecked", "rawtypes" })
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

		/*** 
		 * What if one step has multiple reviewers and
		 * approval from one of those reviewers forward
		 * the request to the next step?
		 * 
		 * version 1.0.0 doesn't have explicit support for this scenario
		 * however, there is a work around. the following example is a
		 * demonstration of that use case. 
		 * ***/
		
		ApprovalWrapper approvalWrapperFlexible = new ApprovalWrapper();
		
		ReviewerModel reviewer01 = new ReviewerModel(1, "R1");
		ReviewerModel reviewer021 = new ReviewerModel(2, "R21");
		ReviewerModel reviewer022 = new ReviewerModel(3, "R22");
		ReviewerModel reviewer03 = new ReviewerModel(4, "R3");
		
		Reviewer<ReviewerModel> rv01 = approvalWrapperFlexible.addReviewer(reviewer01);
		Reviewer<ReviewerModel> rv021 = approvalWrapperFlexible.addReviewer(reviewer021);
		Reviewer<ReviewerModel> rv022 = approvalWrapperFlexible.addReviewer(reviewer022);
		Reviewer<ReviewerModel> rv03 = approvalWrapperFlexible.addReviewer(reviewer03);
		
		Step<Long, Reviewer<ReviewerModel>> step1 = new Step(rv01);
		Step<Long, Reviewer<ReviewerModel>> step21 = new Step(rv021);
		Step<Long, Reviewer<ReviewerModel>> step22 = new Step(rv022);
		Set<Step<Long, Reviewer<ReviewerModel>>> steps = new HashSet<>();
		steps.add(step21);
		steps.add(step22);
		Step<Long, Reviewer<ReviewerModel>> step02 = new Step<>(new Reviewer(-1));
		step02.setSteps(steps);
		Step<Long, Reviewer<ReviewerModel>> step3 = new Step(rv03);

		approvalWrapperFlexible.getSteps().add(step1);
		approvalWrapperFlexible.getSteps().add(step02);
		approvalWrapperFlexible.getSteps().add(step3);

		RequestLifeCycle requestLifeCycle2 = approvalWrapperFlexible.requestProcessLifeCycle();
		System.out.println("\n\n\nrequest life cycle for flexible approval process: " + requestLifeCycle2);
		Optional<Step<?, ?>> optionalStep__ = requestLifeCycle2.getValidatedCurrentStep(rv01);
		if (optionalStep__.isPresent()) {
			Step<?, ?> step = optionalStep__.get();
			step.accept();
			System.out.println("after 1st approval: " + requestLifeCycle2);
			System.out.println("is completed?: " + requestLifeCycle2.isComplete());
		} else {
			if(step1.getSteps() != null) {
				step1.getSteps().forEach(s -> {
					if (s.getReviewer().equals(rv01)) {
						s.accept();
						step1.accept();
						return;
					}
				});
			}
			System.out.println("after 1st approval: " + requestLifeCycle2);
			System.out.println("is completed?: " + requestLifeCycle2.isComplete());
		}
		
		Optional<Step<?, ?>> optionalStep__2 = requestLifeCycle2.getValidatedCurrentStep(rv022);
		if (optionalStep__2.isPresent()) {
			Step<?, ?> step = optionalStep__2.get();
			step.accept();
			System.out.println("after 2nd approval: " + requestLifeCycle2);
			System.out.println("is completed?: " + requestLifeCycle2.isComplete());
		} else {
			if(step02.getSteps() != null) {
				step02.getSteps().forEach(s -> {
					if (s.getReviewer().equals(rv022)) {
						s.accept();
						step02.accept();
						return;
					}
				});
			}
			System.out.println("after 2nd approval: " + requestLifeCycle2);
			System.out.println("is completed?: " + requestLifeCycle2.isComplete());
		}
		
		Optional<Step<?, ?>> optionalStep__3 = requestLifeCycle2.getValidatedCurrentStep(rv03);
		if (optionalStep__3.isPresent()) {
			Step<?, ?> step = optionalStep__3.get();
			step.accept();
			System.out.println("after 3rd approval: " + requestLifeCycle2);
			System.out.println("is completed?: " + requestLifeCycle2.isComplete());
		} else {
			if(step3.getSteps() != null) {
				step3.getSteps().forEach(s -> {
					if (s.getReviewer().equals(rv03)) {
						s.accept();
						step3.accept();
						return;
					}
				});
			}
			System.out.println("after 3rd approval: " + requestLifeCycle2);
			System.out.println("is completed?: " + requestLifeCycle2.isComplete());
		}
	}
}
```
# Things to follow

ReviewerModel class is the class which is considered to be the Reviewer in the real world example. So the ReviewerModel class could be Employee class or User class or Manager class in your project. Just to ensure that the ```hashCode()``` and the ```equals()``` methods are overriden in each of your Reviewer model class.

```java
@Override
public int hashCode() {
	return Objects.hash(id, name); // auto generated hashcode method is fine, you don't need to change that
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
	return id == other.id; // this line is important as the reviewer identity will be validated
	                       // based on its unique id not by any other property, 
	                       // so be sure to understand that, if you have employeeId, or ManagerId, or UserId
			       // as an unique identifier in your project, then this 			           
			       // statement would be employeeId == other.employeeId or managerId == other.managerId like that.
}
```
output:
```
request life cycle: RequestLifeCycle [steps=[Step [t=null, reviewer=Reviewer [reviewer=1], checkList=null, status=PENDING, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=2], checkList=null, status=PENDING, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=3], checkList=null, status=PENDING, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null]], isComplete=false, isExpired=false, remarks=]
after approval by 1L: RequestLifeCycle [steps=[Step [t=null, reviewer=Reviewer [reviewer=1], checkList=null, status=ACCEPTED, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=2], checkList=null, status=PENDING, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=3], checkList=null, status=PENDING, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null]], isComplete=false, isExpired=false, remarks=]
is completed?: false
after approval by 2L: RequestLifeCycle [steps=[Step [t=null, reviewer=Reviewer [reviewer=1], checkList=null, status=ACCEPTED, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=2], checkList=null, status=ACCEPTED, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=3], checkList=null, status=PENDING, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null]], isComplete=false, isExpired=false, remarks=]
is completed?: false
after approval by 3L: RequestLifeCycle [steps=[Step [t=null, reviewer=Reviewer [reviewer=1], checkList=null, status=ACCEPTED, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null], Step [t=null, reviewer=Reviewer [reviewer=2], checkList=null, status=ACCEPTED, date=Mon Jan 23 20:54:50 BDT 2023, approvalType=null, steps=null]], isComplete=true, isExpired=false, remarks=]
is completed?: true
```
# Integration with other programming languages

This library could be compatible with other programming languages also through the provided java support library of the language.
For example, 

* py4J provides java support for python programmers (explore here: https://www.py4j.org/)
* java support for php developers (https://github.com/php-java/php-java)

# What's Next

If you find this library helpful by any chance and want to integrate in your project but need assistance, then let's talk:

linkedin: shorturl.at/gpwD8
