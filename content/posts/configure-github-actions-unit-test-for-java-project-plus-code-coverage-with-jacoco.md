+++
date = '2025-05-15T14:46:00+07:00'
draft = false
title = 'Configure GitHub Actions Unit Test for Java Project (Plus code coverage with Jacoco)'
author = 'ahmad'
+++

In this short article, we will talk about how to set up GitHub Actions for Java projects (such as Spring Boot projects). We also integrate Jacoco to produce code coverage information.

## Configure Jacoco

We must set up Jacoco in our `build.gradle` file. The script is like this :

```plaintext
plugins {
    /* Other plugins */
    id 'jacoco'
}

/* Other configurations and dependencies */

test {
    testLogging {
        events "passed", "failed", "skipped"
    }
    finalizedBy jacocoTestReport
}

jacoco {
    toolVersion = "0.8.9"
}

jacocoTestReport {
    dependsOn test // tests are required to run before generating the report
    reports {
        xml.required = false
        csv.required = true
        html.outputLocation = layout.buildDirectory.dir('jacocoHtml')
    }
}
```

Here are some explanations :

```plaintext
plugins {
    /* Other plugins */
    id 'jacoco'
}
```

We must include Jacoco in `plugins` to use for the code coverage generator for the Java project.

```plaintext
test {
    testLogging {
        events "passed", "failed", "skipped"
    }
    finalizedBy jacocoTestReport
}
```

The `testLogging` part is optional. It's just to show you which test codes are passed, failed, and skipped.

```plaintext
jacocoTestReport {
    dependsOn test // tests are required to run before generating the report
    reports {
        xml.required = false
        csv.required = true
        html.outputLocation = layout.buildDirectory.dir('jacocoHtml')
    }
}
```

In this part, we set up which location to save the code coverage report in HTML pages. We can use this to see the code coverage information when we test the code on our local machine. For the case of GitHub Action, we only use the .csv output for code coverage. The image below is an example of Jacoco output. But, we don't use it for this case.

![Jacoco example](/images/coverage-1.webp)

By default, `jacoco` will save the .csv output in `build/reports/jacoco/test/jacocoTestReport.csv`. We will use this to configure the GitHub Action script in the next step.

## Configure GitHub Action workflow code

```yaml
name: Java CI

on:
  pull_request:
    branches:
      - master

jobs:
  tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Grant execute permissions to Gradle wrapper
        run: chmod +x gradlew

      - name: Validate Gradle wrapper
        uses: gradle/wrapper-validation-action@ccb4328a959376b642e027874838f60f8e596de3

      - name: Run test
        run: ./gradlew test

      - name: Generate JaCoCo Badge
        uses: cicirello/jacoco-badge-generator@v2
        with:
          jacoco-csv-file: build/reports/jacoco/test/jacocoTestReport.csv
```

Here are some explanations :

```yaml
name: Java CI
```

For this case, we name the GitHub Actions with `Java CI`. You can name it whatever you want.

```yaml
on:
  pull_request:
    branches:
      - master
```

The `on:pull_request` specifies when the workflow should be triggered. In this case, we want the script to be triggered on each pull request. In this example, we set it for `master` branch. This means every time developers make a pull request to `master` branch, this workflow will be triggered.

```yaml
jobs:
  tests:
    runs-on: ubuntu-latest
```

The `jobs` section defines the jobs that will run as part of the workflow. In this case, we named the job as `tests`.

`runs-on: ubuntu-latest` specifies the operating system environment in which the job will run. We use Ubuntu's latest version for this example.

```yaml
    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Grant execute permissions to Gradle wrapper
        run: chmod +x gradlew

      - name: Validate Gradle wrapper
        uses: gradle/wrapper-validation-action@ccb4328a959376b642e027874838f60f8e596de3

      - name: Run test
        run: ./gradlew test

      - name: Generate JaCoCo Badge
        uses: cicirello/jacoco-badge-generator@v2
        with:
          jacoco-csv-file: build/reports/jacoco/test/jacocoTestReport.csv
```

The `steps` section contains a list of steps that will be executed as part of the job.

* `uses: actions/checkout@v3`: This step checks out the repository's code, allowing subsequent steps to access and interact with the project files.
    
* `name: Set up JDK 17`: This step sets up JDK 17 on the runner environment. The action `actions/setup-java` is used to do this.
    
* `run: chmod +x gradlew`: This step grants execute permissions to the Gradle wrapper script, allowing it to be executed in the next step.
    
* `name: Validate Gradle wrapper`: This step uses the `gradle/wrapper-validation-action` to validate the Gradle wrapper.
    
* `run: ./gradlew test`: This step runs the `test` task of Gradle using the Gradle wrapper. The `test` task is responsible for executing the tests of the Java project.
    
* `name: Generate JaCoCo Badge`: This step uses the `cicirello/jacoco-badge-generator` action to generate a code coverage badge. The badge is based on the JaCoCo test coverage data stored in the `jacocoTestReport.csv` file, which is typically generated by the Gradle build during the test execution.
    

## Let's give a test

### The Workflow

To test whether your workflow is working or not, you must create a pull request to your GitHub repository. You can follow this [GitHub documentation](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) on how to create a pull request (PR).

If all tests are passed, you will get this notification on your PR :

![PR success](/images/coverage-2.webp)

Otherwise, you will get a failed notification. You can use this information to consider whether you will accept the PR or not.

You can click **Show all checks** to see the details. If you enter the details, you will see some information about the workflow that just ran.

![Show all](/images/coverage-3.webp)

Inside the **Run test** part, you can see the detail about the test execution.

![Run test](/images/coverage-4.webp)

### Code Coverage

To get the code coverage information, Open **your Github repository > Actions**. Then, you will get the list of all previously executed workflows, like this, the list is timely ordered :

![Workflow list](/images/coverage-5.webp)

If you choose one of them, you will get this :

![Workflow detail](/images/coverage-6.webp)

In the **test summary** part, you can see your code coverage.

## Conclusion

Automated unit testing is an important part of CI (Continuous Integration) in the software development life cycle. By utilizing GitHub Actions to handle unit tests, we automate one aspect of CI in our software development.

## Additional Resource

* [Building and testing Java with Gradle](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-java-with-gradle)
    
* [Creating a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)