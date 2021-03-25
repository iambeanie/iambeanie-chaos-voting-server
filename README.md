# iambeanie-chaos-voting-server

# How to setup the server

The server is built in .NET 5 and the client is javascript so in theory it could run on any server / hosting solution. I'm only familiar with windows and IIS so here it is:

## You will need to run a windows machine with the following windows features enabled
Search for Turn Windows Features on or off in the windows search bar
Internet Information Services
- Web Management Tools
  - IIS Management Console
  - IIS Management Scripts and Tools
  - IIS Management Service
- World Wide Web Services (everything) *
- Remote Differential Compression API Support
- Windows Process Activation Service (everything)

With this done you should be able type in inetmgr into the windows search bar and open up IIS Manager, if it opens, YAY. 

Why: IIS is a windows based hosting solution for websites and will make it far easier for us to host our website and have the outside world able to see it. The features we've enabled both turn on IIS and enable features that we need to run the server (ie: web sockets).

## Install the .net hosting bundle

Downloading and install the latest Hosting Bundle for .NET v5, from this page: https://dotnet.microsoft.com/download/dotnet/5.0 . You want the latest Hosting Bundle, which will be on the right hand side, not the links that say x64 or x86, specifically 'Hosting Bundle'. 

Why: This will install the necessary runtimes to run the server application itself, without this, the server wouldn't recognise the code being run.

## Hosting the Server

### Option 1: Self Hosting

If you're determined to host on your own machine, this carries with it some risks. Opening up your own machine as a hosting box is NOT recommended. So anyway, here's how to do it:

#### Forward ports 6970 and 6971 to your machine

This assumes you're running a run of the mill home configuration. If you're not, you're probably already better at this than me <3.

In Windows, open up a cmd prompt. Run 'ipconfig' and take note of your IPv4 address, usually 10.x.x.x or 192.168.x.x.

Open your routers configuration software, usually found by going to the ip address found next to 'Default Gateway' after running ipconfig. Forward ports 6970 and 6971 TCP and UDP to your local computers IP Address. **

Why: We want any communication coming into your network on port 6970 and 6971 to come to your computer, where the server is hosted. Without this step, communication coming those ports will stop at the router.

### Option 2: Hosting on a Server

  1. Obtain a DNS record for the chosen URL
    * You will need a record for both the UI and API, i just use an extra SAN
  2. Obtain an SSL certificate matching the DNS record (lets encrypt is pretty good and free!)

## Create the websites

1. Download the latest build artefacts from this github.
2. Open up IIS through inetmgr.
3. Right click on sites -> Add new Website
4. Point it to the folder 'VotingAPI' from the extracted build artefacts
5. Give it any name really 'Chaos API' will do
6. If you're self hosting, give it port 6970
7. If you're server hosting, put it to https and select the installed SSL Certificate
8. Voting API is now running
9. Right click on sites -> Add new Website
10. Point it to the folder 'VotingUI'
11. Give it any name really 'Chaos UI' will do
12. If you're self hosting, give it port 6971
13. If you're server hosting, put it to https and select the installed SSL Certificate

The server should now be hosted.

For self hosted solutions, grab your external IP address (note: different to the internal one) from any IP revealing site (eg: ipchicken or whatsmyip). On a different computer go to that ip address by typing the following into a browser: http://YOUR-IP-ADDRESS:6970 for the API and http://YOUR-IP-ADDRESS:6971 for the UI. The API should return a 404, this means it is successful and the UI should show something.

For server hosting solutions, visit the URL denoted in the DNS entries.

### Configure the hosted UI

Open the folder where you've put the VotingUI, open up appSettings.js in notepad or similar. Replace the api url with the domain address:

Self Hosted: if your API's address is 'http://187.251.45.10:6970/' then you should put '187.251.45.10:6970' in the file.
Server Hosting: if your API's address is 'https://google.com.au/' then you should put 'google.com.au'.
Note: no trailing slashes

Restart IIS by running 'iisreset' in an admin command prompt.


# Setting up the client

Download the files from this github.

Open the folder 'Put Anywhere and run ChaosClientIntermediary exe', open the file appsettings.json.

Same as under 'Configure the Hosted UI', put the URL of the API here and save. Run the ChaosClientIntermediary.exe, it will show that it's monitoring file changes and connected to the voting server if it worked.

Copy the contents of the 'Put Contents in GTAV Folder' into the root of your GTAV folder. Open up the chaosmod folder, run the configapp.exe and click save. The config app is separate so that new versions of the app don't overwrite your existing config.




# How the server works

The voting server works by hosting a separate web server that clients can connect to and place their votes. Voting options are pushed from the client to the server and then vote results are pushed back to the client through a websocket (no exposed endpoint).

I've taken the shortest dirtiest path to get this working.

Chaos Mod -> Intermediary <-> Server <-> Clients

Chaos Mod modifications:

I've added a heap of new effects (feel free to turn them off).

Each time the timer countdown reaches 0, it writes a file to C:/chaos/voteoptions.txt which contains four voting options with effect id and effect description:

lowgravity,Low Gravity
peds_roasting,Roasting
peds_minigun,Give Everyone A Minigun
tp_lsairport,Teleport To LS Airport

After it has written out the voteoptions, it reads in the result of the previous vote which is stored as an effect id in C:/chaos/vote.txt. ***

Chaos Client Intermediary:

The Intermediary is a .NET 5 console application, it puts a filewatcher on C:/chaos/voteoptions.txt and opens a web socket to the Chaos API. Whenever it detects a change in the vote options file (ie: when the chaos mod writes out new options), it sends the new voting options to the voting server.

Whenever the server sends back an updated vote result through the web socket, it writes out the latest vote result to C:/chaos/vote.txt

Server:

The server manages the voting process. It listens for new voting options, informing the clients as needed.

TLDR: The chaos mod tells the intermediary to tell the server to tell the clients that voting needs to happen.


* Not really, can get away with much less
** Doesn't have to be port 6970, can realistically be anything but 6970 is unused from my experience
*** I'm not proficient with C++ anymore, i've used file writing / reading as the easiest way to move back into C# land, where i am far more familiar. This could easily be done with a websocket in C++ but for the life of me, i can never get anything to build. The API generates a swagger json file and i've actually managed to make it create a C++ client but cannot get it to build.
