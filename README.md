# Higher Diploma in Science in Computing (Software Development) 
# Distributed Systems Project: Application that simulates the operations of a smart automated environment that would support the achievement of one of the UNs SDG’s

We were tasked to develop an application that simulates the operations of a smart automated environment that would support the achievement of one of the UNs SDG’s. My assigned SDG was

*To Ensure inclusive and equitable quality education and promote lifelong learning opportunities for all*

# Service Definitions

I chose to develop an E-learning system based around gRPC which will contain 3 services to perform different functions.

## The functionalities within each service

• **Monitor student attendance**. This will be done via face recognition or RFID cards. Attendance data will be shared with a central system.

• **Student Learning Assistant**. This will track students progress and suggests tailored additional learning materials. It could also be used to generate quizzes and adjust the difficult based on the student’s measured abilities

• **Learning Environment Monitor.** This will track and record several environmental conditions within the classroom such as temperature, CO2 levels, lighting etc. It would then either send alerts to the teacher or automatically adjust the settings.

The services were implemented with 3 .proto files

## SmartAttendanceMonitor.proto

*RPC Definition:*

- recordAttendance(AttendanceRequest) returns (AttendanceResponse)
- Used when a student checks in to a class.

*Example Request:*

studentId: "S0435"

timeStamp = "2025-06-30.09:00:00"

location = "Room 504"

*Example Response:*

Status: "Attendance recorded successfully"

*Parameters Used:*

|  |  |  |
| --- | --- | --- |
| Parameters | Data Type | Description |
| studentId | string | Unique student identifier |
| timeStamp | string | Date and time that attendance was recorded |
| location | string | Location of attendance check in |
| Status | string | Indicates success or failure of attendance being recorded with a message |

*RPC Definition:*

- GetAttendanceSummary(SummaryRequest) returns (SummaryResponse)
- Returns a summary of the student’s attendance.

*Example Request:*

studentId: "S0435"

*Example Response:*

sessionsTotal: 15

sessionsAttended: 13

attendanceRate: 86.67

*Parameters Used:*

|  |  |  |
| --- | --- | --- |
| Parameters | Data Type | Description |
| sessionsTotal | Int32 | Number of session occurrence |
| sessionsAttended | Int32 | Number of sessions the student attended |
| attendanceRate | float | Subsequent attendance rate as a percentage |

## StudentLearningAssistant.proto

#### RPC Definition:

- GetLearningRecommendation(RecommendationRequest) returns (RecommendationResponse)
- Used to return a learning recommendation based on student performance.

#### Example Request:

studentId: "S0435"

subject: "Quadratic Equations"

performanceScore: 89.5

#### Example Response:

recommendedSubject: " Polynomials "

resources: ["Polynomials.mp4", " Polynomials \_worksheet.pdf", " Polynomials\_quiz.html"]

#### Parameters Used:

|  |  |  |
| --- | --- | --- |
| **Parameters** | **Data Type** | **Description** |
| subject | string | Subject the student is currently studing |
| performanceScore | Int32 | A score indicating the students performance (0-100) |
| recommendedSubject | string | The next topic recommended to the student by the system |
| resources | string | Recommended resources to help the student further their studies |

#### RPC Definition:

- SubmitAssessment(AssessmentRequest) returns (AssessmentResponse)
- Submits completed assessment data and returns the result.

#### Example Request:

studentId: "S0435"

assessmentId: "A101"

assessmentAnswers: ["A", "C", "A", "E"]

#### Example Response:

performanceScore: 75

passed: true

feedback: "Good work! Please review question 2"

#### Parameters Used:

|  |  |  |
| --- | --- | --- |
| **Parameters** | **Data Type** | **Description** |
| assessmentId | string | Unique identifier for assessment being submited |
| assessmentAnswers | string | Answers given for the assessment |
| performanceScore | Float | Calculated score for the assessment |
| Passed | Boolean | Calculated pass or fail based on assessment criteria |
| feedback | String | Generated comment on performance |

## LearningEnvironmentMonitor.proto

*RPC Definition:*

- GetRoomConditions(RoomConditionsRequest) returns (RoomConditionsResponse)
- Used to retrieve real-time data about the room conditions.

*Example Request:*

location = "Room 504"

*Example Response:*

temperature: 21.0

humidity: 35

co2Level: 800

airQuality: "Good"

*Parameters Used:*

|  |  |  |
| --- | --- | --- |
| Parameters | Data Type | Description |
| temperature | float | Real time Measured temperature (degrees centigrade) |
| humidity | float | Real time Measured humidity (%) |
| co2Level | float | Real time Co2 measure(ppm) |
| airQuality | string | Calculated description of air quality |

*RPC Definition:*

- SetThresholds(ThresholdRequest) returns (ThresholdResponse)
- Used to define or update the minimum and maximum safe threshold of the environment conditions.

*Example Request:*

location = "Room 504"

temperatureMin: 18

temperatureMax: 25

humidityMax: 60

co2Threshold: 1000

*Example Response:*

message: "Thresholds updated successfully"

status: true

*Parameters Used:*

|  |  |  |
| --- | --- | --- |
| Parameters | Data Type | Description |
| TemperatureMin | float | Sets the minimum temperature (degrees Centigrade) |
| temperatureMax | float | Sets the minimum temperature (degrees Centigrade) |
| humidityMax | float | Sets the Maximum humidity level (%) |
| co2Threshold | float | Sets threshold of acceptable CO2 levels (ppm) |

# Service Implementations

A java class was made for each service

- SmartAttendanceMonitorService.java
- LearningEnvironmentMonitorService.java
- StudentLearningAssistantService.java

Each one extends its own interface,

- SmartAttendanceMonitorImpBase
- LearningEnvironmentMonitorImpBase
- StudentLearningAssistantImpBase

They contain the methods detailed above. Each method also contains basic input validation logic. This could be expanded upon in the future.

At the end of each service class, we use responseObserver.onNext() and onCompleted() to send the response to the client and complete its use

# Naming Services.

Each service registers itself using jmDNS at the start of the file. It declares the service type using \_smartattendance.\_tcp.local.This is all held within a try/catch block to handle exceptions

Example:

![Image: image_003](./Distributed%20Systems%20CA%20-%20REPORT%20-%20Matthew%20Warren_images/image_003.png)

A class called ServiceDiscoveryClient.java discovers the published services with addServiceListener(). This means It resolves services dynamically. This allows results in high flexibility and dynamic service discovery.

# Remote Error Handling & Advanced Features

There is basic validation included for all inputs. The validation is implemented within the service methods. This Validation could be improved in the future and extended but this was deemed out of the scope of this project.

Example of validation:

![Image: image_004](./Distributed%20Systems%20CA%20-%20REPORT%20-%20Matthew%20Warren_images/image_004.png)

This approach ensures that the server can handle by returning error messages to the client.

# Client - Graphical User Interface (GUI)

The GUI is an important part of the system, allowing users to interact with all 3 services. A gui file was created – ControllerGUI.java. This was made using Java swing package. It provides input fields, lables and buttons for each servicelaid out in a vertical fashion. It also features a output text box to display results.

The GUI communicates with each gRPCservice using the blocking generated from the proto files. Each button triggers a gRPC call and the response is then displayed.

# GitHub Repo

The project was developed from the start using version control in GitHub – Link below.

Link: <https://github.com/astro-mat/NCI_DistSysCA>