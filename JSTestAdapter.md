## Seamlessly running JavaScript tests with VSTest
VSTest provides tremendous extensibility for running any kind of unit tests and functional tests in a pipeline. A single test runner to rule them all. To count a few examples, NUnit, XUnit and MSTest. Be it running in Visual Studio, CLI or using VSTest task in azure pipelines. With just a few simple arguments and a few clicks and voila! Your build is passing (or failing).

With all this push to open source and SaaS services and the ever evolving UX design, there is a heavy requirement for JavaScript testing. JavaScript with its plethora of test frameworks and countless number of test runners, there still doesn’t seem to be a way to seamlessly run JavaScript tests in the pipeline. The only JavaScript test adapter that comes close to leveraging this extensibility is Chutzpah, but while chutzpah provides a way to run JavaScript tests with VSTest it is limited to running only unit tests through PhantomJS as its runtime.

#### JSTest to the rescue for your functional test deprived developers 
JSTestAdapter is an open source test adapter extension for VSTest for running, well you guessed it, any kind of JavaScript tests. With its high extensibility for JavaScript test frameworks and environments, JSTest runs JavaScript with node at its core.

The adapter currently has support for Node as its runtime environment, two major test frameworks, Jasmine and Mocha. It also supports running tests through Jest, so if you can’t leverage extensibility of the adapter for test frameworks and environments, you can utilize Jest. Running tests with JSTest is same as running tests with any other adapter extension of VSTest, whether it be via CLI, Visual Studio or VSTest tasks in Azure Devops. VSTest gets passed a test adapter path, 

#### Usage

Let's take an example of a basic calculator test.

```javascript
// Calculator.js
function add(a, b) {
    return a + b;
}

module.exports = {
    add
}
```

```javascript
// CalculatorTest.js
const assert = require("assert");
const calculator = require("./calculator.js")

describe("Calculator tests", () => {
    
    it("This test will fail", () => {
        assert.equal(calculator.add(1, 2), 2);
    });

    it("This test will pass", () => {
        assert.equal(calculator.add(1, 2), 3);
    });

});
```

And now to setup `jasmine` and `jstestadapter` npm packages.

```powershell
> npm install jasmine
> npm install jstestadapter
```

Running this test is as simple as passing the path to the node module and the path to the test file as arguments.

```powershell
> vstest.console.exe --TetsAdapterPath:.\node_modules\jstestadapter .\calculatortest.js
```

![Imgur](https://i.imgur.com/bwrEbDJ.png)

Since the default test framework for JSTest is jasmine, let's try to run the tests with mocha. 

```powershell
> vstest.console.exe --TetsAdapterPath:.\node_modules\jstestadapter .\calculatortest.js -- JSTest.TestFramework=mocha
```

`... -- JSTest.TestFramework=mocha` is how run-settings are provided to vstest. Run setting configurations can also be defined in an xml and passed as a cli argument to vstest.console.exe, `--Settings:runsettings.xml`. Run-settings can also contain configurations specific to the test framework/runner in question. Now let's try this with jest, since jest uses file patterns to check for test files, package.json is used as the test file container for running tests through jest.

```powershell
vstest.console.exe --Settings:RunSettings.xml --TestAdapterPath:.\node_modules\jstestadapter .\package.json
```

With RunSettings.xml:
```xml
<RunSettings>
    <JSTest>
        <TestFramework>jest</TestFramework>
        <TestFrameworkConfigJson>{
            "collectCoverage": true,
            "verbose": true
        }</TestFrameworkConfigJson>
    </JSTest>
</RunSettings>
```

You can find more options for jest at https://jestjs.io/docs/en/configuration.html

#### Running javascript tests in Azure Pipelines

We can use the VSTest task in Azure Pipelines to run javascript tests with VSTest and JSTest. Let's go ahead and create a pipeline and add VSTest task to it.

![Imgur](https://i.imgur.com/wVinSKh.png)

We configure the task with runsettings.xml and a test match pattern for javascript files

![Imgur](https://i.imgur.com/gsDeQcX.png)

#### Limitations

While the adapter was designed to be cross-platform there are still a few issues with JSTestAdapter when trying to run it with .NET core build for VSTest. Along with not smoothly being able to run on linux (there are a few workarounds to get it to work), even though the architecture is designed to be runtime and test framework abstracted but currently there is no public API to leverage that extensible design.

#### Conclusion

JSTestAdapter provides a fantastic way of running any kinds of tests through node in a pipeline setting like Azure Devops Pipeline. With support for the latest trend in javascript testing and node runtime, it makes it possible to run about any kind of tests. The javascript tests for the adapter itself are run through the adapter. Checkout the [_testception_](https://dev.azure.com/karanjitsingh/JSTestAdapter/_build/results?buildId=108&view=results) in action for yourself.

###### Source: [JSTestAdapter Repo](https://github.com/karanjitsingh/JSTestAdapter/)
