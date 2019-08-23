# Fidelity automation test framework
#### This tool is developed for the purpose of Fidelity.

### Main goal:
- Desktop testing (Microsoft office Word plugin)
- - - -
### Сontents:

**1.** Test environment preparation

**2.** The instructions of usage

**3.** Test framework structure

**4.** Additional tools 

**5.** Additional recommendations
- - - -
### Test environment preparation:
**1.**  Install java JDK as a code language (version: 1.8.0) [link to download java JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- 1.1 set jdk path to 'JAVA_HOME' environment variable
- 1.2 add %JAVA_HOME%\bin to 'Path' environment variable
- - - -
**2.** Install MAVEN as a project builder (version: 3.5.4) [link to download maven](http://maven.apache.org/download.cgi)
- 2.1 set maven path to 'MAVEN_HOME' environment variable
- 2.2 add %M2_HOME%\bin to 'Path' environment variable
- - - -
**3.** Install MS Office 365 using link: [link to download and install Office 365](https://www.office.com/?auth=2) with plugin templates (carnally tests can run just in AQA environment)
- - - -
**4.** Install WinAppDriver:
- download driver: [link to download WinAppDriver](https://github.com/Microsoft/WinAppDriver/releases)
- run WinAppDriver.exe from the installation directory (E.g. C:\Program Files (x86)\Windows Application Driver) to be clear the installation is done 
- turn on the developer mod into your machine (computer): Settings -> Update & Security -> For developer section -> check on the Developer mod
- be sure you add WinAppDriver to the environment variables 'path' because of @BeforeSuite - use it wherever it needs
- - - -
**5.** Additional Microsoft Word configuration:
- open MWord -> File -> Options -> Save -> uncheck: 'Save AutoRecover information' checkbox
- open MWord -> File -> Options -> General -> uncheck: 'Show the Start screen when this application starts'
- - - -
**6.** ```mvn clean install -U -DskipTests=true``` - run this command from the start to ensure that all project dependencies installed and up to date
- - - -
### The instructions of usage:
#### 1. Be low script commands allow to run tests in different way:
##### All tests :

```mvn clean test -Dsuite=all_tests -P 'cv' or 'fil'```

- - - - 
##### Particular suite:
Currently evaluable suites are: 
- based_smoke (hes not completed yet)
- smoke_rating
- smoke_general
- rating_note
- general_note
- all_tests

Currently evaluable profiles are (list of appropriate configuration e.g. credentials, different links, ech...):
 - 'cv' - for CoreValue needs
 - 'fil' - for Fidelity needs

```mvn clean test -Dsuite='name of your particular suite' -P 'cv' or -P 'fil'```
- - - -
##### Single test:

```mvn clean test -Dtest='pakage name of test class'.'class name' -P 'cv' or -P 'fil' -DfailIfNoTests=false```
- - - -
##### Additional run information:
In order for a run with token authorization to be succeed use following:
- command with run script:
  - Dapi.corp.id.value="Your corpId value" (first header value)
  - Dapi.token.value="Your token value" (second header value)
e.g.: ```mvn clean test -Dsuite=all_tests  -Dapi.corp.id.value="Your corpId value" -Dapi.token.value="Your token value" -P fil```

or
- make your own property configuration into .pom xml file:
  - project based dir -> pom.xml -> find tag: <api.corp.id.value>empty_by_default</api.corp.id.value> and put your corp id value instate of 'empty_by_default'
  - project based dir -> pom.xml -> find tag: <api.token.value>empty_by_default</api.token.value> and put your token value instate of 'empty_by_default'
e.g.: ```mvn clean test -Dsuite=all_tests -P fil```
- - - -
#### 2. Report generation:
After tests run use Allure report generator: 
- ```mvn allure:report``` - to get it into base project directory (the command generates a package *allure-result* into the based framework directory and allow to lunch report manually via IDE or other tool which can render HTML document)

- ```mvn allure:serve``` - to get it on your localhost (will open automatically in your browser)
- - - -
#### 3. CI: 
As continuous integration tool was chosen JENKINS:

Configured Jenkins job is running all tests every night by scheduled trigger.

And every morning we have a chance to analise the tests report following links:

- [link to CI setup](https://filcrm.atlassian.net/wiki/spaces/IRA/pages/557219880/Automation+CI)
- [link to get all Allure reports](http://10.10.0.30:82/allure/)
- [link to get Fidelity CI job](http://10.10.0.30:8080/view/Fidelity/job/FIDELITY_REPORT/)
- - - -
### Test framework structure:
#### 1. Packages structure:
Test framework was build as multi modules project and include such modules:
- - - -
**1.** **fidelity_main** - module include packages with the main code structure:
 - **api** - package for work with API services include: 
   - *client* - package with code, which creates connection and make requests to needed endpoints, e.g. Entity, User, ech...
   - *controls* - package with code, which except API responses from the clients, can pars them, and transfer needed data in the tests
   - *helpers* - package with code, which is wrapper above controls, need for avoid code duplication
 - **component_object** - such package is "Page Object pattern", it means that every application component rendered like an appropriate object/class with methods and logic inside according to application business logic:
   - *document* - package with code, which related with word document pages, components and objects
   - *plugin* - package with code, which related with plugin pages, components and objects
 - **constants** - package include immutable objects e.g. strings like name of topics, ech...
 - **domains** - package include models of some objects or JSON files which are using for serialization and deserialization (converting objects and files into java objects and vise versa)
 - **repos** - realization of previous abstract domain models
 - **enums** - package with specific classes which are like a models of some application small objects at ones, need for some searching, filtering, ech..., to avoid code duplication
- - - -
**2.** **fidelity_test** - module include packages with the automation tests structure:
 - **fidelity_word** - package with tests related with word plugin testing:
   - **general_note** - package with tests related with "General Note Template":
     - *credit* - package with tests related with "General Credit Topic"
     - *equity* - package with tests related with "General Equity Topic"
     - *intelligence* - package with tests related with "General Intelligence Topic"
     - *ipo* - package with tests related with "General IPO Topic"
     - *macro* - package with tests related with "General Macro Topic"
   - **rating_note** - package with tests related with "Rating Note Template":
     - *credit* - package with tests related with "Rating Credit Topic"
     - *equity* - package with tests related with "Rating Equity Topic"
     - *shorting* - package with tests related with "Rating Shorting Topic"
 - **test_runners** - classes with logic of driver creation and session starting
 - **resources** - package which includes additional files and files .properties:
   - **config.properties** - file includes the main project configuration, that's like command point of all project, such file directly mapped to the existing in the pom.xml profiles: 'cv' and 'fil'
   - **environment.properties** - file includes environment test information, e.g. plugin, JDK and platform versions ech...
   - **log4j.properties** - file includes default *'log4j'* logger configuration
   - **attachments** - package which includes files need to be attached by tests
   - **images** - package which includes files/images need for Sikuly library using
   - **suites** - package which includes .xml files/test suites
- - - -
**3.** **fidelity_util** - module include packages with the util classes/static helpers
 - - - -
**4.** **pom.xml** - a *Project Object Model* or *POM* is the fundamental unit of work in Maven. It is an XML file that contains information about the project and configuration details used by Maven to build the project. It contains default values for most projects
- - - -
**5.** **.gitignore** - It's a list of files you want git to ignore in your work directory, file specifies intentionally untracked files that Git should ignore. Files already tracked by Git are not affected
- - - -
**6.** **README.md** - file contains information about other files in a directory, otherwise it is the automation project documentation in our case
- - - -
#### 2. Libraries using:
**1.** **TestNG** - is a testing framework inspired from JUnit and NUnit but introducing some new functionality that makes it more powerful and easier to use

**2.** **AssertJ** - is a library that provides a fluent interface for writing assertions. Its main goal is to improve test code readability and make maintenance of tests easier

**3.** **Log4j** - is a reliable, fast and flexible logging framework

**4.** **Selenium** - is a free (open source) automated testing suite for web applications across different browsers and platforms. Selenium is not just a single tool but a suite of software's, each catering to different testing needs of an organization

**5.** **Lombok** - is a java library that automatically plugs into your editor and build tools, spicing up your java. Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more

**6.** **Allure** - is a flexible lightweight multi-language test report tool that not only shows a very concise representation of what have been tested in a neat web report form, but allows everyone participating in the development process to extract maximum of useful information from everyday execution of tests

**7.** **Vavr** - is a functional library for *Java 8+* that provides immutable data types and functional control structures

**8.** **SikuliX** - is a scripting application used to automate anything that can be seen on the screen of your desktop computer running Windows, Mac or some Linux/Unix

**9.** **RestAssured** - is a library that provides a domain-specific language (DSL) for writing powerful, maintainable tests for RESTful APIs

**10.** **Jackson** -  is a high-performance JSON processor for Java. Its developers extol the combination of fast, correct, lightweight, and ergonomic attributes of the library. Jackson provides many ways of working including simple POJO converted to/from JSON for simple cases. Jackson provides a set of annotations for mapping too

**11.** **JavaFaker** - is a library that can be used to generate a wide array of real-looking data from addresses to popular culture references
- - - -
### Additional tools which need to be installed:
- IDE, we recommend to use IntelliJ IDEA (free version: *Community*) for tests development needs [link to download and install](https://www.jetbrains.com/idea/download/#section=windows)
- Lombok Plugin (plugin needs to be installed, annotation processing needs tobe launched)
- Markdown support (plugin needs to get a proper *Readme.md*/documentation file view)
- .xml manifest installation:

**1**. Follow [link](https://docs.microsoft.com/en-us/office/dev/add-ins/publish/host-an-office-add-in-on-microsoft-azure) and please do the first two steps

**2**. Follow the [link](https://docs.microsoft.com/en-us/office/dev/add-ins/testing/create-a-network-shared-folder-catalog-for-task-pane-and-content-add-ins) and please do all the steps, file manifest should be added into folder which is shown in the current link

**3**. From the root of project please execute ```npm run build:sit```

**4**. The folder named *dist* will be generated 

**5**. Everything what is inside this folder please copy to [https://insight-authoring-ui-sit.npapps.paas.bip.uk.fid-intl.com](https://insight-authoring-ui-sit.npapps.paas.bip.uk.fid-intl.com)

**6**. It should form the next URL [index.html](https://insight-authoring-ui-sit.npapps.paas.bip.uk.fid-intl.com/index.html)

**7**. After following this link you should see the grey page
- - - -
### Additional recommendations which need to be followed (because of desktop testing specific):
**1**. While tests running the session must be untouchable - any user action (e.g. mouse moving, windows changing) might impact run

**2**. The language while run, should be 'English US' - test suite hes a few cases (attachment, image reading, when path to, is setting) when driver switch to desktop, it is immediately influenced by language used on machine (computer)

**3**. To get particular branch with framework version you need:

```git clone -b <branch> <remote_repo>``` (e.g.: git clone -b 1.17.0 https://kostyantins@bitbucket.org/FILCRM/authoring-ui-automation.git)
- - - -
