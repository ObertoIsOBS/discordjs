# My Discord Documentation (Made Easy)
This is the documentation for my Discord Bot.
After lots of expireience I have found alot about Discord.js.
The modules I use in this bot you can find in the index.js file.
## Simple Command Handler

Basically instead of putting everthing in our index.js file we put it in there own seperate files. This is alot more organized and easier to manage. To do this we in summary tell the bot where our commands are. We wont tell it what to do with them yet.
**Make sure you make a folder called commands in your main directory!!**
Then use this code to setup your `index.js` file.
```
//Call the required packages and files.
const Discord = require('discord.js');
const client = new Discord.Client();
const {Client, RichEmbed} = require('discord.js');
const fs = require("fs");
const ms = require("ms");
const Enmap = require('enmap');
const eco = require('discord-economy');
// This is so we can use this package anywhere.
client.eco = eco;

// This creates a new map with our commands
client.commands = new Enmap();
// This reads your commands file
fs.readdir("./commands/", (err, files) => {
  if (err) return console.error(err);
  files.forEach(file => {
  // This will ignore any files that dont end with .js  
    if (!file.endsWith(".js")) return;
    let props = require(`./commands/${file}`);
   // This slices of the .js and leaves us with the command name
   let commandName = file.split(".")[0];
    // Log in the console that the commands are loaded
    console.log(`Loaded ${commandName}`);
    client.commands.set(commandName, props);
  });
});
// Log the bot with your bot token.
client.login('Your token'); 
```
At the moment the bot wont do anything but load the commands. We need to setup an event before it can do anything. But first we want to change how we log the bot. Right now were using `client.login('Your Token')` to log the bot. We dont want to do that so lets create a config file.
## Simple Config File

A config file is just a file that will hold all the information we need for our bot.
We will create a simple config file, we'll name this `config.json` and save it in our bots main directory.
Since this is a JSON file you can use this template for making your config file.
```
{
     "token": "Your Token",
     "prefix": "!"
}
```
*As you can see I also added a prefix section, we will use that later.*
now we need to define our config file in `index.js`. To do this we will use call it with the other pages like this 
`const config = require('./config.json);` Then we will now log the bot like this. `client.login(config.token);` See? Isnt that much better. Now at this point our bot still does not do anything lets fix that by create an event.
Also in our code we will redfine config as `client.config = config;` This allows us to use the file anywhere.
## Creating a simple event handler

Basically what were doing is telling the bot what to do when something happens.
We will do this similar to how we did it with the command handler. So you will need to make a folder called events.
Next we need to make a program to handle the events like we did with commands. Use this code in your `index.js` file. 
*We will put this above `client.commands = new Enmap();` in our code.*
```
// This reads the evnts folder
fs.readdir("./events/", (err, files) => {
    if (err) return console.error(err);
    files.forEach(file => {
      // This gets the events
      const event = require(`./events/${file}`);
      // Removes .js from the event name
      let eventName = file.split(".")[0];
      // Tells the client to listen to this event
      client.on(eventName, event.bind(null, client));
    });
  });
```  
Now we have an event handler but we still have no events. :(
We need to add some events.
## First Event

The first event we will add is the "ready" event. This is emitted everytime the client starts.
For this you will need to make a file called `ready.js` in the events folder you created. Then use the following code.
```
// First we get the client from "index.js"
module.exports = (client) => {
// then we log in the console "Logged in as Your bot name"
    console.log(`Logged in as ${client.user.tag}!`);
    // Log that the bot has started
    console.log('Bot Started');
       // Set our client presence to "Playing with Java Script!"
       client.user.setActivity("with Javascript!"); 
// end of module
}
```
Now everytime your bot starts it will run this code but, still no commands. :(
Lets set that up next.
## The Message Event
Next we will create the message event. This emits everytime a message is sent.
This is how the commands will run because all commands will be recieved from this event.
```
module.exports = (client, message) => {
    // Ignore all messages from bots
    if (message.author.bot) return;
  
    // Ignores messages that dont contain our prefix
    if (message.content.indexOf(client.config.prefix) !== 0) return;
  
    // Defines the args and command
    const args = message.content.slice(client.config.prefix.length).trim().split(/ +/g);
    const command = args.shift().toLowerCase();
  
    // Gets the commands from our Enmap
    const cmd = client.commands.get(command);
  
    // If the command doesn't exist exit
    if (!cmd) return;
  
    // Run the command
    cmd.run(client, message, args);
  };
```
Now we can use commands.
## First Command

The first command we will make will be a "ping" command. This will respond mentioning you and saying pong! and the ping speed of the bot. First we will make a file in our commands folder called `ping.js` Then use the following code.
```
// call the discord.js package
const Discord = require("discord.js");
// get nesscary packages from our files
module.exports.run = async (client, message, args) => {
// !ping
// Replies with pong!"ping speed" in ms
    message.reply(`pong! \`${client.pings.length}ms\``);
}
// end of module
```
Now when we type !ping our bot will respond with pong! and the pingspeed.
Now that we've added a command we can see how other commands must be formatted. You can use this template for any commands you want to make.
```
const Discord = require("discord.js");
module.exports.run = async (client, message, args) => {
    const config = client.config;

    // do something
}
```
Now that you can make simple commands we will now go over some other events.
***This will begin my documentation of discord.js events***.
## "guildMemberAdd" event

This event is emitted whenever a new member joins a guild.
```
/*
@param member - The member that joined.
*/
```
Example:
```
// get the required packages
module.exports = (client, member) => {
// get the channel to send the message to by name
const channel = member.guild.channels.find(ch => ch.name === "member-log");
// send the message with the guild name and mention the member
channel.send(`Welcome to **${member.guild}** ${member}!!`);

// end of module
}
```
This uses 3 things that we have not learned yet. The first is getting a channel from a guild. To do this we use `member.guild.channels.find(ch => ch.name === "member-log");` I'll break this down for you. the fist thing is `member.guild` this gets the guild of the member that just joined. Next, we use `channels.find` This seraches all cahnnels in the guild we just found. Then, we use `(ch => ch.name === "member.log");` This tells the client which channel to find. We tell it to find the channel "member-log". The next thing used in the message that we sent was `member.guild` which we also just used to get the guild of the Member. This is the same thing. It collects the name of the guild. The last thing is `member` which we have used twice. We defined this in the module exports. This gives you the members name and mentions them.
