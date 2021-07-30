# CloudSlang in a Docker Container

## Minimal Configuration

THIS IS THE CONTAINER YOU WANT.

The 'minimal' install of CloudSlang is enough to run all CloudSlang automations.  The intention of this container is to offer a minimal install of CloudSlang (CS) with updated CS content. This minimal install of CS includes:

* The CS CLI (and content)
* The CS WebApp (for REST API)
* Java (headless)
* SSH (Client)

In addition to providing the services listed above, the use case for this container container is to stay up and running, able to start automations via command-line or REST API.

### Running This Container

To start this container, use the following steps.

1. Build the CS image.
   ```
   cd /path/to/Dockerfile
   docker build -t cloudslang:latest .
   
   [optionally, you can pass build args to run newer or older versions]
   
   docker build -t cloudslang:latest --build-arg centos_version=latest --build-arg cs_cli_version=2.0.205 .
   ```
2. Run the CS container.
   ```
   docker run -d -p 8443:8443 --name cloudslang cloudslang:latest
   ```

### Testing This Container via Command-Line

If your container is running successfully, you'll be able to use these commands and get the expected results:

##### Test CS from outside the container using 'docker exec':
```
# docker exec -it cloudslang cslang run --f /opt/cslang-cli/content/io/cloudslang/base/print/print_text.sl --i text=Hi
Loading..
Hi
Operation: print_text finished with result: SUCCESS
Jul 06, 2020 5:39:17 PM org.springframework.shell.core.AbstractShell handleExecutionResult
INFO: Execution id: 100800001, duration: 0:00:09.889
```

##### Test CS from inside the container at the command-line:
```
# docker exec -it cloudslang bash
# cslang run --f /opt/cslang-cli/content/io/cloudslang/base/print/print_text.sl --i text=Hi
Loading..
Hi
Operation: print_text finished with result: SUCCESS
Jul 06, 2020 5:41:51 PM org.springframework.shell.core.AbstractShell handleExecutionResult
INFO: Execution id: 100800001, duration: 0:00:09.887
```

##### Test CS from inside the container using the CloudSlang CLI:
```
# docker exec -it cloudslang bash
# cslang
Loading..
_________ .__                   .____________.__
\_   ___ \|  |   ____  __ __  __| _/   _____/|  | _____    ____    ____
/    \  \/|  |  /  _ \|  |  \/ __ |\_____  \ |  | \__  \  /    \  / ___\
\     \___|  |_(  <_> )  |  / /_/ |/        \|  |__/ __ \|   |  \/ /_/  >
 \______  /____/\____/|____/\____ /_______  /|____(____  /___|  /\___  /
        \/                       \/       \/           \/     \//_____/
2.0.205
Welcome to CloudSlang. For assistance type help.
cslang> run --f /opt/cslang-cli/content/io/cloudslang/base/print/print_text.sl --i text=Hi
Hi
Operation: print_text finished with result: SUCCESS
Execution id: 100800001, duration: 0:00:02.088
```

### Testing via the REST API

Get the version of the CS WebApp:
```
# curl -k --user admin:admin -X GET https://localhost:8443/cs/rest/version
{"version":"2.0.205"}
```

Get the 'inputs' for the CS 'print_text' flow:
```
# curl -k --user admin:admin -X GET https://localhost:8443/cs/rest/flows/v1/io.cloudslang.base.print.print_text/inputs
[{"name":"text","required":true,"sensitive":false,"default":null}]
```

Run a flow using CS via REST, and return the execution ID:
```
# curl -k --user admin:admin -X POST -H 'Content-Type: application/json' -d '{ "slangFlowId":"io.cloudslang.base.print.print_text","runInputs":{"text":"Hi"},"systemProperties": {} }' https://localhost:8443/cs/rest/v1/executions
100000001
```

Get the status of an execution that was run:
```
# curl -k --user admin:admin -X GET https://localhost:8443/cs/rest/v1/executions/100000001
{"executionId":100000001,"status":"COMPLETED","result":"SUCCESS","outputs":"{}"}
```

See the output of the 'print_text' flow:
```
# docker logs cloudslang
Hi
```

Using username:password for these automations is a bit insecure. Using basic authentication isn't much better, but it's something. To use basic authentication tokens instead, use one of these commands:
```
# echo "author:author" | base64
RaNDomSTRiNg==
# curl -v -k --user author:author -X GET https://localhost:8443/cs/rest/version

[ Look specifically for this line: ]
> Authorization: Basic RaNDomSTRiNg==
```

Replace the '--user username:password' part of the command line with an HTTP header for all CS REST API calls. For example:
```
# curl -k -X GET -H "Authorization: Basic RaNDomSTRiNg=" https://localhost:8443/cs/rest/version
{"version":"2.0.205"}
```

NOTE: The REST API for CS comes with preinstalled self-signed certs.  So username and password aren't exactly sent in plain text.  However, it also includes the ability to import and use your own certificates.  In addition, the accounts 'admin' and 'author' are default accounts. You can create custom accounts with custom permissions (READ or READ/WRITE) in this file here:

* /opt/cslang-rest/security/users.yml

### Importing New CS Content

Micro Focus (MF) creates new content for CS regularly, and posts it for free on the MF Marketplace. This free CS content can be found here:

* <https://marketplace.microfocus.com/ITOM/search?q=cloudslang>

NOTE: You can create an account for free on the MF Marketplace, and then download the CS content you like.

This Dockerfile was designed to automatically pull in any content you download from the MF Marketplace. To do this, perform these steps:

1. Open a web browser, and navigate to the link above.
2. Download any content you'd like.
   * CS comes with Base content, but there's usually an updated Base content pack on the MF Marketplace. At a minimum, that's a good one to grab.
3. Create a directory at the same place as your Dockerfile, called 'mf_content'.
4. Copy your downloaded content (jar files) to the 'mf_content' directory.
5. Uncomment the COPY line in this Dockerfile below.
   * From this:
   ```
   #COPY ./mf_content /media/mf_content
   ```
   * To this:
   ```
   COPY ./mf_content /media/mf_content
   ```
6. Follow the steps above to build, run, and test this container (with new content).
