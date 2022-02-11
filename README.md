Page Object model is an object design pattern in Selenium, where web pages are represented as classes, and the various elements on the page are defined as variables on the class. All possible user interactions can then be implemented as methods on the class:

clickLoginButton();
setCredentials(user_name,user_password);
Since well-named methods in classes are easy to read, this works as an elegant way to implement test routines that are both readable and easier to maintain or update in the future. For example:

In order to support Page Object model, we use Page Factory. Page Factory in Selenium is an extension to Page Object and can be used in various ways. In this case we will use Page Factory to initialize web elements that are defined in web page classes or Page Objects.

Web page classes or Page Objects containing web elements need to be initialized using Page Factory before the web element variables can be used. This can be done simply through the use of initElements function on PageFactory:

LoginPage page = new LoginPage(driver);
PageFactory.initElements(driver, page);
Or, even simpler:

LoginPage page = PageFactory.intElements(driver,LoginPage.class)
Or, inside the web page class constructor:

public LoginPage(WebDriver driver) {           
         this.driver = driver; 
         PageFactory.initElements(driver, this);
}
Page Factory will initialize every WebElement variable with a reference to a corresponding element on the actual web page based on configured “locators”. This is done through the use of @FindBy annotations. With this annotation, we can define a strategy for looking up the element, along with the necessary information for identifying it:

@FindBy(how=How.NAME, using="username")
private WebElement user_name;
Every time a method is called on this WebElement variable, the driver will first find it on the current page and then simulate the interaction. In case we are working with a simple page, we know that we will find the element on the page every time we look for it, and we also know that we will eventually navigate away from this page and not return to it, we can cache the looked up field by using another simple annotation:

@FindBy(how=How.NAME, using="username")
@CacheLookup
private WebElement user_name;
This entire definition of the WebElement variable can be replaced with its much more concise form:

@FindBy(name="username")
private WebElement user_name;
The @FindBy annotation supports a handful of other strategies that make things a bit easier:

id, name, className, css, tagName, linkText, partialLinkText, xpath
@FindBy(id="username")
private WebElement user_name;


@FindBy(name="passsword")
private WebElement user_password;


@FindBy(className="h3")
 private WebElement label;


@FindBy(css=”#content”)
private WebElement text;
Once initialized, these WebElement variables can then be used to interact with the corresponding elements on the page. The following code will, for example:

user_password.sendKeys(password);
… send the given sequence of keystrokes to the password field on the page, and it is equivalent to:

driver.findElement(By.name(“user_password”)).sendKeys(password);
Moving on, you will often come across situations where you need to find a list of elements on a page, and that is when @FindBys comes in handy:

@FindBys(@FindBy(css=”div[class=’yt-lockup-tile yt-lockup-video’]”)))
private List<WebElement> videoElements;
The above code will find all the div elements having two class names “yt-lockup-tile” and “yt-lockup-video”. We can simplify this even more by replacing it with the following:

@FindBy(how=How.CSS,using="div[class=’yt-lockup-tile yt-lockup-video’]")
private List<WebElement> videoElements;
Additionally, you can use @FindAll with multiple @FindBy annotations to look for elements that match any of the given locators:

@FindAll({@FindBy(how=How.ID, using=”username”),
	@FindBy(className=”username-field”)})
private WebElement user_name;
Now that we can represent web pages as Java classes and use Page Factory to initialize WebElement variables easily, it is time we see how we can write simple Selenium tests using the Page Object pattern and Page Factory.
  
  Simple Selenium Test Automation Project in Java
  Visit www.toptal.com

Click on the “Apply As A Developer” button

On Portal Page first check if it’s opened

Click on the “Join Toptal” button

Fill out the form

Submit the form by clicking on “Join Toptal” button

Setting Up a Project
Download and install Java JDK

Download and install InteliJ Idea

Create a new Maven project

Link “Project SDK” to your JDK, e.g.: on Windows “C:\Program Files\Java\jdkxxx”

Setup groupId and artifactId:

<groupId>SeleniumTEST</groupId>
<artifactId>Test</artifactId>
Add dependencies Selenium and JUnit Maven in your project POM file
   <dependencies>
        <!-- JUnit -->         
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Selenium -->

        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-firefox-driver</artifactId>
            <version>${selenium.version}</version>
        </dependency>

        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-support</artifactId>
            <version>${selenium.version}</version>
        </dependency>

        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>${selenium.version}</version>
        </dependency>

    </dependencies>
Replace Selenium version and JUnit Version with latest version numbers that can be found by searching for JUnit Maven on Google and on Selenium site.

At this point, if auto build is enabled, dependencies should start downloading automatically. If not, just activate Plugins > install > install:install under the Maven Projects panel on the right side of your IntelliJ Idea IDE.
  
  Once the project has been bootstrapped, we can start creating our test package under “src/test/java”. Name the package “com.toptal”, and create two more packages under it: “com.toptal.webpages” and “com.toptal.tests”.
  
  We will keep our Page Object/Page Factory classes under “com.toptal.webpages” and the test routines under “com.toptal.tests”.

Now, we can start creating our Page Object classes.

HomePage Page Object
The very first one we need to implement is for Toptal’s homepage (www.toptal.com). Create a class under “com.toptal.webpages” and name it “HomePage”.

Determining Element Locators
  On Toptal’s homepage we are interested about one element in particular, and that is the “Apply as a Developer” button. We can find this element by matching the text, which is what we are doing above. While modeling web pages as Page Object classes, finding and identifying elements can often become a chore. With Google Chrome or Firefox’s debugging tools, this can be made easier. By right clicking on any element on a page, you can activate the “Inspect Element” option from the context menu to find out detailed information about the element.

One common (and my preferred) way is to find elements using Firefox’s FireBug extension, in combination with Firefox web driver in Selenium. After installing and enabling FireBug extension, you can right click on the page and select “Inspect element with FireBug” to open FireBug. From the HTML tab of FireBug, you can copy the XPath, CSS Path, Tag name or “Id” (if available) of any element on the page.
  
  By copying the XPath of the element in the screenshot above, we can create a WebElement field for it in our Page Object as follows:

@FindBy(xpath = "/html/body/div[1]/div/div/header/div/h1")
WebElement heading;
Or to keep things simple, we can use the tag name “h1” here, as long as it uniquely identifies the element we are interested in:

@FindBy(tagName = "h1")
WebElement heading;
DeveloperPortalPage Page Object
Next, we need a Page Object that represents the developer portal page, one that we can reach by clicking on the “Apply As A Developer” button.

On this page, we have two elements of interest. To determine if the page has loaded, we want to verify the existence of the heading. And we also want a WebElement field for the “Join Toptal” button.

DeveloperApplyPage Page Object
And finally, for our third and last page object for this project, we define one that represents the page containing developer application form. Since we have to deal with a number of form fields here, we define one WebElement variable for every form field. We find each field by their “id” and we define special setter methods for every field that simulate keystrokes for the corresponding fields.
  
  Writing a Simple Selenium Test
With Page Object classes representing our pages, and user interactions as their methods, we can now write our simple test routine as a series of simple method calls and assertions.
  
