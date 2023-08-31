# test
test repo for git command demostration
package IRDAI;

import java.awt.AWTException;
import java.awt.Robot;
import java.awt.event.KeyEvent;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.time.Duration;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.util.Date;
import java.util.NoSuchElementException;
import java.util.Scanner;
import java.util.Set;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import Public_Announcement.XLUtility;
import io.github.bonigarcia.wdm.WebDriverManager;

public class publicAnnouncement {
	
	
	static LocalDate fromDate = LocalDate.now();
	LocalDate toDate = LocalDate.now();
	static LocalTime currentTime = LocalTime.now();
    static LocalTime newTime = currentTime.plusMinutes(1);
	
	static DateTimeFormatter dateformatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
	static DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("HH-mm-ss");
	private static String excelFileName;
	
	 
	
	private static final String LOG_FilePath = "./Logs/Public_Announcement_" + dateformatter.format(fromDate) + "_" + timeFormatter.format(newTime) + "_log.txt";
	  
	
	
	public static void writeLog(String message) {
	     String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
	    
	    String logEntry = timeStamp + " - " + message;

	    try (BufferedWriter writer = new BufferedWriter(new FileWriter(LOG_FilePath, true))) {
	        writer.write(logEntry);
	        writer.newLine();
	        System.out.println("Log written successfully: " + logEntry);
	    } catch (IOException e) {
	        System.err.println("Error writing log: " + e.getMessage());
	    }
	}
	
	public static void main(String[] args) throws Exception {
		 
		writeLog("---Start Execution---");
		
		Scanner s =new Scanner(System.in);
		System.out.println("--AGI Brains--");
		 	System.out.println("Please Enter The Time In 24-Hour Format (E.g--> 13,00)");
		
		String timeInput = s.next();
		if (timeInput.matches("\\d{2},\\d{2}")) {
			
            scheduleScriptExecution(timeInput );
        }
		 else {
			 writeLog("Entered time is invalid :"+timeInput );
			  System.out.println("Enter a Valid Time");
		 }
		
		
	}
		public static void scheduleScriptExecution( String timeInput  ) {
		 
		String[] target= timeInput.split(",");
		int hours = Integer.parseInt(target[0]);
		int minutes = Integer.parseInt(target[1]);
	     LocalTime currentTime = LocalTime.now();
        final LocalTime targetTime = LocalTime.of(hours, minutes);
       
        LocalDateTime targetDateTime = LocalDateTime.of(LocalDate.now(), targetTime);
        if (currentTime.isAfter(targetTime)) {
            // If the current time is after 12:00 PM, schedule it for the next day
            targetDateTime = targetDateTime.plusDays(1);
        }
        long initialDelayInSeconds = LocalDateTime.now().until(targetDateTime, ChronoUnit.SECONDS);

        // Schedule the task to run at 12:00 PM every day
        ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
        executor.scheduleAtFixedRate(() -> {
        	//System.setProperty("webdriver.chrome.driver","./drivers/chromedriver.exe");
        	WebDriverManager.chromedriver().setup();
    		WebDriver driver = new ChromeDriver ( );
            try {
            	
            	writeLog("----Call webscrapping method----"  );
                webScrapping2(driver,targetTime );
                //writeLog("----Call Pdf Download method----" );
                //PdfDownload(driver, targetTime);
                writeLog("---Find the missing Files---");
                 
                
                writeLog("----Browser closed----");
               // driver.close();
            } catch (Exception e) {
                e.printStackTrace();
                writeLog("Exception :"+e );
            }  
        }, initialDelayInSeconds, Duration.ofDays(1).getSeconds(), TimeUnit.SECONDS);
        
        
        
    }
		
		
public static void webScrapping2(WebDriver driver ,LocalTime time ) throws IOException, InterruptedException, AWTException {
	 
	LocalTime targetTime = time;
	 
	driver.manage().timeouts().implicitlyWait(5,TimeUnit.SECONDS);
	
	driver.manage().window().maximize();
	
	
	 
	 
	writeLog("---ChromeDriver was started successfully---" );
	 	LocalDate fromDate = LocalDate.now();
    LocalDate toDate = LocalDate.now();
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("HH-mm-ss");
    for (LocalDate date = fromDate; !date.isAfter(toDate); date = date.plusDays(1)) {
         String  formattedDate = date.format(formatter);
          String formattedTime = targetTime.format(timeFormatter);
        
         
        excelFileName ="./data/Public_Announcement.xlsx";
        
	    XLUtility xlutil = new XLUtility(excelFileName);
   
           
       driver.get("https://ibbi.gov.in/en/public-announcement?ann=&title=&date="+formattedDate);
 	 
	//write header in the excel sheet
	 xlutil.setCellData("Public_Announcement", 0, 0,"Type of PA");
	 xlutil.setCellData("Public_Announcement", 0, 1,"Date Of Announcement");

	 xlutil.setCellData("Public_Announcement", 0, 2,"Last date of Submission");
	 xlutil.setCellData("Public_Announcement", 0, 3,"Name of Corporate Debtor");
	 xlutil.setCellData("Public_Announcement", 0, 4,"Name of Applicant");
	 xlutil.setCellData("Public_Announcement", 0, 5,"Name of Insolvency Professiona");
	 xlutil.setCellData("Public_Announcement", 0, 6,"File Name");

	 xlutil.setCellData("Public_Announcement", 0, 7,"Remarks");
	 
	 int lastRow2 = xlutil.getRowCount("Public_Announcement");
	 lastRow2++;
	String For_DateOF_Data = formattedDate;
  xlutil.setCellData("Public_Announcement", lastRow2, 3,"---"+For_DateOF_Data+"---");
// capture table rows
	 
	WebElement table = driver.findElement(By.xpath("//*[@id=\"block-ibbi-content\"]/div/div/div[2]/table"));
	 
  int rows=driver.findElements(By.xpath("//*[@id=\"block-ibbi-content\"]/div/div/div[2]/table/tbody/tr")).size();
  
 System.out.println(rows);
 String source = driver.getPageSource();
 
  try {
	  if (source.contains("No Result Found"))
	  {
		 String Error = "--No Result Found--"+formattedDate;
		  int lastRow1 = xlutil.getRowCount("Public_Announcement");
			 lastRow1++;
		  xlutil.setCellData("Public_Announcement",lastRow1,4,Error);
           continue;
	  }
  }
  catch (NoSuchElementException e)
  {
	System.err.println("Exception handle.."+e);
	writeLog("Exception Caught :"+e);
  }
  
  for (int i=1;i<=rows;i++)
  {
	  
	  String TypeofPA = table.findElement(By.xpath("tbody/tr["+i+"]/td[1]")).getText();
  	 String Date_Of_Announcemen = table.findElement(By.xpath("tbody/tr["+i+"]/td[2]")).getText();
	 String Last_date_of = table.findElement(By.xpath("tbody/tr["+i+"]/td[3]")).getText();
	 String Name_of_Corporate = table.findElement(By.xpath("tbody/tr["+i+"]/td[4]")).getText();
	 String Name_of_Applicant = table.findElement(By.xpath("tbody/tr["+i+"]/td[5]")).getText();
	 String Name_of_Insolvency = table.findElement(By.xpath("tbody/tr["+i+"]/td[6]")).getText();
	 
	  
	 //getting the Pdf file name in the link  
	WebElement PdfFilename = table.findElement(By.xpath("tbody/tr["+i+"]/td[7]/a"));
	 String file = PdfFilename.getAttribute("onclick");
	  String Public_Announcement = file.substring(file.lastIndexOf("/")+1);
	  int indexOfPdf = Public_Announcement.indexOf(".pdf");
	  if (indexOfPdf != -1)
	  {
		  Public_Announcement = Public_Announcement.substring(0, indexOfPdf + 4);
	  }
	   Public_Announcement = Public_Announcement.replace("%20"," ").replace(":", "_").replace("')", " ").replace(";", " ");
	  
	  System.out.println("Pdf No:"+i+"--->"+Public_Announcement);
	 String Remarks = table.findElement(By.xpath("tbody/tr["+i+"]/td[8]")).getText();
	
	 int lastRow = xlutil.getRowCount("Public_Announcement");
	 lastRow++;
	  
	  
  xlutil.setCellData("Public_Announcement",lastRow ,0, TypeofPA);
  xlutil.setCellData("Public_Announcement",lastRow ,1, Date_Of_Announcemen);
  xlutil.setCellData("Public_Announcement",lastRow ,2, Last_date_of);
  xlutil.setCellData("Public_Announcement",lastRow ,3, Name_of_Corporate);
  xlutil.setCellData("Public_Announcement",lastRow ,4, Name_of_Applicant);
  xlutil.setCellData("Public_Announcement",lastRow ,5, Name_of_Insolvency);
  xlutil.setCellData("Public_Announcement",lastRow ,6, Public_Announcement );
  xlutil.setCellData("Public_Announcement",lastRow ,7, Remarks);
  writeLog("Public announcement data : "+TypeofPA +Date_Of_Announcemen+Last_date_of+Name_of_Corporate+Name_of_Applicant+Name_of_Insolvency+Public_Announcement+Remarks);
 
  
  }
  System.out.println("Data inserted succesfully.>>>>>");
  writeLog("---Data inserted succesfully---"+formattedDate );
  
  //writeLog("----Call Pdf Download method----" );
   PdfDownload(driver, targetTime);
  
	}
	}

 
public static void PdfDownload (WebDriver driver,LocalTime time ) throws InterruptedException, AWTException
{
	writeLog("---Start Pdf Downloading---" );
	WebElement table = driver.findElement(By.xpath("//*[@id=\"block-ibbi-content\"]/div/div/div[2]/table"));
	int rows = driver.findElements(By.xpath("//*[@id=\"block-ibbi-content\"]/div/div/div[2]/table/tbody/tr")).size();
      System.out.println(rows);
      
      
	for (int i = 1; i <= rows; i++) {
		String source = driver.getPageSource();
		 
		  try {
			  if (source.contains("No Result Found"))
			  {
				  writeLog("---No Result Found---" );
				  continue;
			  }
		  }
		  catch (NoSuchElementException e)
		  {
			System.out.println("Exception handle.."+e);
			writeLog("Exception caught:"+e );
		  }
	    WebElement PdfFilename = table.findElement(By.xpath("tbody/tr[" + i + "]/td[7]/a"));
	    PdfFilename.click();
	    Thread.sleep(2000);

	    String oldwindow = driver.getWindowHandle();
	    Set<String> newwindow = driver.getWindowHandles();

	    for (String handle : newwindow) {
	        if (!handle.equals(oldwindow)) {
	            driver.switchTo().window(handle);

	            WebDriverWait wait = new WebDriverWait(driver, 10);
	            wait.until(ExpectedConditions.jsReturnsValue("return document.readyState === 'complete';"));

	             Thread.sleep(1000);
	            Robot rt = new Robot();
	            rt.keyPress(KeyEvent.VK_CONTROL);
	            rt.keyPress(KeyEvent.VK_S);
	            rt.keyRelease(KeyEvent.VK_CONTROL);
	            rt.keyRelease(KeyEvent.VK_S);
	            Thread.sleep(1000);
	            rt.keyPress(KeyEvent.VK_ENTER);
	            rt.keyRelease(KeyEvent.VK_ENTER);
	            Thread.sleep(3000);

	            
	            driver.close();
	            
	            System.out.println("Document Downloded.."+i);
	            driver.switchTo().window(oldwindow);
 	        }
	    }
 	}
	writeLog("--Page Completed--" );
	 System.out.println("---Page Completed---");
	 
	  

}
}


 
