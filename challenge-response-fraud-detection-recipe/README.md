![Quine Logo](https://uploads-ssl.webflow.com/61d5ee2c68a4d5d61588037b/61e6e6ab0e87d1f27094489b_Quine%20Logo.svg)

Quine recipe for ingesting logs from a [Challenge-Response-based](https://www.techtarget.com/searchsecurity/definition/challenge-response-system) Security Verification system (like CAPTCHA) and creating a streaming graph with alerting as a result of [Standing Queries](https://docs.quine.io/components/writing-standing-queries.html) matches that trigger based on derived categorical data.

## Simulated Architecture

![](https://user-images.githubusercontent.com/99685020/174155084-3a5e993a-538e-45d2-a908-f754284d5f54.png)

## Setup

This demo is architected to be able to be run on a single system with four (4) pre-requisites:

1.  [Running Quine;](https://docs.quine.io//getting-started/quick-start.html) minimum version 1.2.0
2.  The [fraud.yml](https://github.com/thatdot/ChallengeResponseFraudDetection/blob/main/fraud.yml) recipe file;
3.  The [fraud.json](https://github.com/thatdot/ChallengeResponseFraudDetection/blob/main/fraud.json) sample data file; and
4.  A locally running webhook receiver like https://github.com/adnanh/webhook.

![](https://user-images.githubusercontent.com/99685020/174157059-e7e42e4d-94a8-45ea-805f-d8d25c84602b.png)

## Operation

Run the recipe with a specific version of quine with the following command:

```shell
java -jar quine-x.x.x.jar -r fraud.yml
```

**NOTE:** The fraud.yml recipe specifies `fraud.json` as the ingest file:

```
ingestStreams:
  - type: FileIngest
    path: fraud.json
```

This means that the `fraud.json` file needs to be in the same directory as Quine, or you will need to edit the `path` entry in the `fraud.yml` file with the full path of the location of `fraud.json`.

This will create a streaming graph from the 25,000 record sample data file (fraud.json) consisting of:

*   customer -> user -> event
*   user -> email
*   event -> ip address
*   event -> device
*   event -> cookie
*   event -> minute

![](https://user-images.githubusercontent.com/99685020/173842067-648de872-c8d6-40e8-a217-bfcd1864419b.png)

### Counters

There are counters on the nodes for:

*   IP addresses
*   Devices
*   Day/hour/minute

### First/Last Seen

There are first seen/last seen dates for:

*   Cookie
*   IP address
*   Device
*   Email
*   User

### Quick Queries

There are Quick Queries on the appropriate nodes to utilize edge counts for:

*   Count of connected Events by Customer IP address
*   Connected Events by Customer
*   Count of connected Customers by IP address
*   List of Customers that Used this IP address
*   IP addresses by frequency
*   Email addresses by customer
*   Devices by user
*   Cookies by user chronologically

### Alarms

There is a standing query that emits a POST to a local webhook listener.  This will require two things:

1 - Running a webhook receiver on bare metal like the binaries provided here [https://github.com/adnanh/webhook/releases/latest](https://github.com/adnanh/webhook/releases/latest;), or in a Docker container like this [https://github.com/adnanh/webhook-contrib/tree/master/docker](https://github.com/adnanh/webhook-contrib/tree/master/docker); and  
2 - Updating the recipe configuration to match the IP address and port of your webhook receiver.

### Sample Queries

Finally, there are Sample Queries that show **one node** of each type to facilitate exploration of the data and Quick Queries.

## Sample Data

The sample log data (1,000 lines) for this recipe is generated in a [Mockaroo](https://www.mockaroo.com/projects/33591) project.  
(Reach out to Allan if you need access to the project.)

*   People [schema](https://www.mockaroo.com/08802da0)
*   Customer [schema](https://www.mockaroo.com/663de710)
*   Enforcement challenge [schema](https://www.mockaroo.com/affe89a0)
*   Log file [schema](https://www.mockaroo.com/c2f00130)

Generate data using cURL with the following command:

```shell
curl "https://api.mockaroo.com/api/c2f00130?count=1000&key=8893c2a0" > "fraud.json"
```

**Note:** You may adjust the number of entries in the `fraud.json` file by adjusting the value for the `count` parameter to any integer between 1 and 5000.

e.g., to generate 3,000 lines of data, issue the following command:

```shell
curl "https://api.mockaroo.com/api/c2f00130?count=3000&key=8893c2a0" > "fraud.json"
```
