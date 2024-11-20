# OS-Demo
A demonstration of deadlocls and priority inversion
SIMULATION OF PRIORITY INVERSION IN A HOME AUTOMATION SYSTEM USING ARDUINO UNO AND WOKWI 

SLOT L49-L50
OPERATING SYSTEMS
BCSE303P


Group Members:

Varshini S. Sakthivel 23BCE1805
I Naveen Abraham 23BCE1836
Sowmiya V 23BCE1452
Riya Renju 23BCE1290

Priority Inversion
Priority inversion occurs when a higher-priority task is forced to wait because a lower-priority task is holding a resource that it needs. This can significantly degrade system performance, especially in real-time systems.
Effects: The primary effect is that high-priority tasks are delayed by low-priority tasks, which can lead to missed deadlines or degraded system responsiveness. In extreme cases, priority inversion can cause critical tasks to fail, resulting in severe system malfunctions.
Mitigation Techniques: Techniques like priority inheritance (where the lower-priority task temporarily inherits the priority of the blocked higher-priority task) or priority ceiling (where tasks run at a higher priority while holding certain resources) are commonly used to mitigate priority inversion.

Deadlock
A deadlock is a situation where a set of tasks are unable to proceed because each is waiting for a resource held by another in the set. This creates a circular wait, where no task can release resources and none can make progress.

The 4 Types of Deadlocks:
Mutual Exclusion: At least one resource is non-shareable.
Hold and Wait: Tasks hold resources while waiting for others.
No Preemption: Resources cannot be forcibly taken from a task.
Circular Wait: There is a closed chain of tasks, each waiting for a resource held by the next.
Mitigation Techniques: Deadlocks can be avoided using deadlock prevention (modifying resource allocation policies), deadlock avoidance (e.g., Banker’s algorithm), or deadlock detection and recovery (monitoring and intervening when deadlocks occur).

The Simulation:
In this project, a home automation system is simulated with three priority levels of tasks, represented by three LEDs. Each priority handles different aspects:
Low Priority: Temperature logging.
Medium Priority: Detecting temperatures.
High Priority: Security system (responding to disturbances).
Deadlock Scenario: Triggered by a sudden spike in temperature.
Priority Inheritance: To show mitigation.

Results
Case 1: Normal Run
Low Priority: Executed. Logs temperature (temperatureLow).
Medium Priority: Executed. Measures temperature (temperatureMedium).
High Priority: Executed. Activates security system.

Case 2: Deadlock
Low Priority: Executed. Begins logging temperature (sharedTemperature).
Medium Priority: Waits for the shared variable (sharedTemperature), which is in use.
High Priority: Waits for low priority to release the shared resource (sharedTemperature).

Case 3: Priority Inversion
Low Priority: Executed. Logs temperature (sharedTemperature) first.
High Priority: Waits while low priority holds the shared resource (sharedTemperature).
Medium Priority: Executed after high priority task. Measures temperature.

Case 4: Priority Inheritance
Low Priority: Executed. Temporarily inherits higher priority to complete quickly.
High Priority: Executed after low priority releases the shared resource.
Medium Priority: Executed after high priority. Measures temperature.

The Code: 
#include <DHT.h>
//parts
#define DHTPIN 2
#define DHTTYPE DHT22
#define LED_LOW 8
#define LED_MEDIUM 9
#define LED_HIGH 10
DHT dht(DHTPIN, DHTTYPE);


//prio initialisation
enum Priority{
  PRIORITY_LOW,
  PRIORITY_MEDIUM,
  PRIORITY_HIGH
};


//prio variables
float temperatureLow = 0.0;  
float temperatureMedium = 0.0;
float sharedTemperature = 0.0;


//starting setup
void setup(){
  Serial.begin(9600);
  dht.begin();


  pinMode(LED_LOW, OUTPUT);
  pinMode(LED_MEDIUM, OUTPUT);
  pinMode(LED_HIGH, OUTPUT);


  digitalWrite(LED_LOW, LOW);
  digitalWrite(LED_MEDIUM, LOW);
  digitalWrite(LED_HIGH, LOW);
}


//overall case loop
void loop(){
  runCase(1);
  delay(5000);
  runCase(2);
  delay(5000);
  runCase(3);
  delay(5000);
  runCase(4);
  delay(5000);
  Serial.println("All cases executed.");
}


void runCase(int caseNumber){
  Serial.print("Running Case ");
  Serial.println(caseNumber);
 
  switch (caseNumber) {
    case 1:
      normalRun();
      break;
    case 2:
      deadlockRun();
      break;
    case 3:
      priorityInversionRun();
      break;
    case 4:
      priorityInheritanceRun();
      break;
    default:
      Serial.println("Invalid case number");
  }
}


void normalRun(){
  Serial.println("Normal Run - All priorities execute correctly.");


  temperatureLow = dht.readTemperature();
  Serial.print("Low priority temperature: ");
  Serial.println(temperatureLow);
  digitalWrite(LED_LOW, HIGH);
  delay(500);
  digitalWrite(LED_LOW, LOW);


  temperatureMedium = dht.readTemperature();
  Serial.print("Medium priority temperature: ");
  Serial.println(temperatureMedium);
  digitalWrite(LED_MEDIUM, HIGH);
  delay(500);
  digitalWrite(LED_MEDIUM, LOW);


  digitalWrite(LED_HIGH, HIGH);
  delay(500);
  digitalWrite(LED_HIGH, LOW);
}


void deadlockRun() {
  Serial.println("Deadlock Run - Shared variable leads to deadlock.");


  //low prio uses shared
  sharedTemperature = dht.readTemperature();
  Serial.print("Low priority starts using shared variable: ");
  Serial.println(sharedTemperature);
  digitalWrite(LED_LOW, HIGH);
  delay(500);


  //high and medium wait
  Serial.println("Medium and high priority tasks waiting for shared variable.");
  digitalWrite(LED_MEDIUM, HIGH);
  digitalWrite(LED_HIGH, HIGH);


    delay(2000);


  //lights off
  digitalWrite(LED_LOW, LOW);
  digitalWrite(LED_MEDIUM, LOW);
  digitalWrite(LED_HIGH, LOW);
}




void priorityInversionRun() {
  Serial.println("Priority Inversion Run - Low priority blocks high priority.");


  sharedTemperature = dht.readTemperature();
  Serial.print("Low priority temperature: ");
  Serial.println(sharedTemperature);
  digitalWrite(LED_LOW, HIGH);
  delay(2000); //low happens for longer, so high waits
  digitalWrite(LED_LOW, LOW);


  Serial.println("High priority task waiting for low priority.");
  digitalWrite(LED_HIGH, HIGH);
  delay(500);
  digitalWrite(LED_HIGH, LOW);


  Serial.println("Medium priority task executing.");
  float temperature = dht.readTemperature();
  Serial.print("Temperature: ");
  Serial.println(temperature);
  digitalWrite(LED_MEDIUM, HIGH);
  delay(500);
  digitalWrite(LED_MEDIUM, LOW);
}


void priorityInheritanceRun() {
  Serial.println("Priority Inheritance Run - Low priority inherits high priority to release shared resource.");


  //low starts and inherits high prio
  sharedTemperature = dht.readTemperature();
  Serial.print("Low priority (elevated) temperature: ");
  Serial.println(sharedTemperature);
  digitalWrite(LED_LOW, HIGH);
  delay(1000); //temp elevation of low
  digitalWrite(LED_LOW, LOW);


  //high executes after low is done
  Serial.println("High priority task executing after priority inheritance.");
  digitalWrite(LED_HIGH, HIGH);
  delay(500);
  digitalWrite(LED_HIGH, LOW);


  //medium at the end
  Serial.println("Medium priority task executing.");
  float temperature = dht.readTemperature();
  Serial.print("Temperature: ");
  Serial.println(temperature);
  digitalWrite(LED_MEDIUM, HIGH);
  delay(500);
  digitalWrite(LED_MEDIUM, LOW);
}


The Circuit:



Scope: 

Educational Use
Learning OS Concepts: Demonstrates real-world applications of operating system concepts like deadlock, priority inversion, and scheduling. Practical Demonstration: Useful for teaching resource management and synchronization issues, making it an excellent tool for students learning embedded systems or OS concepts.

Home Automation Systems
Task Management Simulation: Represents how priority-based tasks could be managed in a home automation environment (e.g., controlling HVAC, lighting, security). Failure Scenarios Testing: Shows how deadlock or priority inversion might lead to failures, helping developers create more robust and reliable home automation solutions.

Embedded Systems Design
Real-World Issue Resolution: Highlights common issues in embedded systems where multiple components need shared resources, and improper handling can lead to performance degradation. Algorithm Testing: Provides a platform to experiment with various scheduling algorithms to resolve priority inversion or prevent deadlocks.

Industrial Automation and IoT Applications
Resource Management in IoT Devices: Useful for IoT devices and industrial automation systems where task priority is critical (e.g., safety mechanisms should have the highest priority over regular monitoring). Real-Time System Design: Demonstrates the importance of avoiding deadlocks and ensuring high-priority tasks are executed in real-time scenarios.

Optimization Strategies
Mutex and Semaphore Implementation: Could be extended to implement actual solutions like mutexes or priority inheritance to prevent deadlock and priority inversion. Performance Metrics Analysis: Allows for the measurement of delays, which helps in developing better resource management strategies.

Extension Opportunities
User Interaction: Adding more sensors or user interactions to demonstrate complex scenarios. Priority Inheritance Protocol: Implement priority inheritance or ceiling protocols to demonstrate real-world solutions for priority inversion. Fault Recovery Mechanisms: Add fault detection and recovery mechanisms for handling deadlocks, showing a complete lifecycle of error detection and resolution.

Research and Development
Improving Real-Time Systems: Can be used as a research project to propose improved scheduling algorithms that handle priority inversion or deadlock more efficiently. System Testing and Validation: Serves as a basis for testing how different operating systems or custom schedulers might handle concurrency and resource allocation.

Project Links:
https://wokwi.com/projects/410698784514008065
https://www.canva.com/design/DAGSffc43Uw/wrsHDaEdLzKUzLmv0AI3eg/edit


Bibliography: 
https://www.geeksforgeeks.org/introduction-of-deadlock-in-operating-system/
https://www.geeksforgeeks.org/priority-inversion-what-the-heck/
https://www.sciencedirect.com/topics/computer-science/priority-inversion





