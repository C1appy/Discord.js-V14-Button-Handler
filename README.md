To get started make a new file in your handler file or wherever you wish this to be.

**Paste this code below into said file**

```js
const { resolve } = require("path");
const { readdir } = require("fs").promises;
const { EmbedBuilder } = require("discord.js");
const chalk = require("chalk");
module.exports = (client) => {
  async function getFiles(dir) {
    const dirents = await readdir(dir, { withFileTypes: true });
    const files = await Promise.all(
      dirents.map((dirent) => {
        const res = resolve(dir, dirent.name);
        return dirent.isDirectory() ? getFiles(res) : res;
      })
    );
    return Array.prototype.concat(...files);
  }

  async function run() {
    const files = await getFiles("./Buttons/");
    const jsFiles = files.filter((f) => f.split(`.`).pop() === "js");
    if (jsFiles.length <= 0)
      return console.log(
        chalk.yellowBright.bold(`[Button-Handler] No loadable Buttons detected`)
      );
    console.log(
      chalk.blueBright.bold(
        `[Button-Handler]: Loaded ${jsFiles.length} Buttons`
      )
    );

    jsFiles.forEach((fileName, index) => {
      let props = require(fileName);
      client.Buttons.set(props.custom_id, props);
    });
  }
  run();

  client.on(`interactionCreate`, async (interaction) => {
    if (interaction.isButton()) {
      var cmd = client.Buttons.get(interaction.customId);
      if (cmd)
      if (!interaction.member.permissions.has(cmd.userPermissions || [])) {
        const UserNoPerms = new EmbedBuilder()
          .setColor("Red")
          .setDescription(
            `You Require **${cmd.userPermissions}** to use this button`
          );
        return interaction.reply({
          embeds: [UserNoPerms],
          ephemeral: true,
        });
      } else if (
        !interaction.guild.members.me.permissions.has(cmd.botPermissions || [])
      ) {
        const BotNoPerms = new EmbedBuilder()
          .setColor("Red")
          .setDescription(
            `I Require **${cmd.botPermissions}** to execute this interaction`
          );

        return interaction.reply({
          embeds: [BotNoPerms],
          ephemeral: true,
        });
      } else {
        cmd.run(client, interaction);
      }
    }
  });
};
```

After this is done go to your main file (normally index.js) and paste this somewhere

`require(./Handlers/Buttons)(client);`

**Make sure to change** `./Handlers/Buttons` **to the path where you have it**

**Make sure to add this into your index.js**

`client.Buttons = new Collection();`

Once done make a new folder in your main directory named `Buttons` and then make folders/files for each button
all you need to do is add your buttons customID, user/bot permissions into there and the code to be executed once triggered

**Example of using the button handler**

```js
module.exports = {
  custom_id: "Put your button ID here",
  userPermissions: ["USER PERMISSIONS GO HERE"],
  botPermissions: ["BOT PERMISSIONS GO HERE"],

  run: async (client, interaction) => {

    //all of your code goes in here

  },
};
```
