# ⚡️⚡️⚡️Lightning ⚡️⚡️⚡️

### This is #Reckless, please use it on testnet for now as it is brand new

This document explains how to use Fully Noded to remotely connect to and control your c-lightning node via a Tor V3 hidden service.

First you need to install c-lightning, to do that follow [this](https://github.com/ElementsProject/lightning/blob/master/doc/INSTALL.md) guide for whichever OS you are on.

If you have c-lightning running already you can stop it. `cli/lightning-cli stop`

You will need Tor running on the node. On linux `sudo apt install tor` works, on macOS `brew install tor` does the trick.

Boot tor as a service:
mac `brew services start tor`
Linux: `systemctl start tor`

Create a hidden service that we will use later:
macOS:
`nano /usr/local/etc/tor/torrc` (if it does not yet exist duplicate the torrc.sample in the same directory and delete the .sample extension, saving it as a new file)

On Linux:
`nano /etc/tor/torrc`

Ensure you uncomment these lines:
`ControlPort 9051`
`CookieAuthentication 1`

and find the hidden services section:
```
############### This section is just for location-hidden services ###

## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.
```

Below it add the hidden service we will use to control our lightning node:
```
HiddenServiceDir /usr/local/var/lib/tor/lightning/
HiddenServiceVersion 3
HiddenServicePort 1312 127.0.0.1:1312
```
`ctlr x` > `y` > `return` to save the changes and quit nano test editor

You will then need to create the hidden service directory:
`mkdir /usr/local/var/lib/tor/lightning/`

On linux assign the owner (brew should do this automatically on macOS):
`chown -R debian-tor:debian-tor /usr/local/var/lib/tor/lightning/`

On both linux and mac:
`chmod 700 /usr/local/var/lib/tor/lightning/`

Restart Tor
macOS `brew services restart tor`
linux `systemctl restart tor`

Ensure all went well by running:
`cat /usr/local/var/lib/tor/lightning/hostname`
If it prints something like `ndfiuhfh2fu23ufh21u3bfd.onion` then all is well, if not message me on the Fully Noded Telegram and I can help (maybe).

Save the above hostname, you will need it soon!

If you haven't created a lightning config file now is a good time to do that.
```
cd /home/you/.lightning
nano config
```
paste in the following config which is ideally suited to Fully Noded usage:
```
alias=FullyNoded
network=testnet
log-file=/home/you/.lightning/lightning.log
plugin=/home/you/.lightning/plugins/c-lightning-http-plugin/target/release/c-lightning-http-plugin
log-level=debug:plugin
bind-addr=127.0.0.1:9735
proxy=127.0.0.1:9050
announce-addr=theHostnameYouJustSavedFromThePreviousSteps.onion:1312
http-pass=aPassWordYouWillSoonCreate
http-port=1312
```

Create a directory for the plugins we need:
`mkdir /home/you/.lightning/plugins/`

Download the plugin:
`git clone https://github.com/Start9Labs/c-lightning-http-plugin.git`

Compile the plugin (it's built in Rust so first install Rust):
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
Close the terminal and reopen it so that the Rust command `cargo` is automatically added to your `path`
```
cd /home/you/.lightning/plugins/c-lightning-http-plugin
cargo build --release
```
When it finishes building give it permissions:
`chmod a+x /home/you/.lightning/plugins/c-lightning-http-plugin/target/release/c-lightning-http-plugin`

Start lightning:
`cd /home/you/lightning`
`./lightningd/lightningd`

In Fully Noded go to "Settings" > "Node Manager" > ⚡️:
- add the rpcuser: `lightning`
- add the rpc password which is the `http-pass` you added to the config from above: `aPassWordYouWillSoonCreate`
- add the onion address (hostname) we created earlier, in the config it is the value for `announce-addr`, ensure you add the port at the end: `theHostnameYouJustSavedFromThePreviousSteps.onion:1312`

Thats it, Fully Noded will now automatically use those credentials for any lightning related functionality. You can only have one lightning node at a a time, to add a new one just overwrite the existing credentials.

In Fully Noded you will see lightning bolt ⚡️ buttons in a few places, tap them to see what they do.

For invoices, Fully Noded by default creates on-chain invoice's from your Bitcoin Core node, tap the lightning bolt to create a lightning invoice, you can add optional amount (btc) and label for the bolt11 invoices.

Fully Noded no longer differentiates between hot and cold balances. Your balances now shows an on-chain 🔗 balance and a lightning ⚡️ balance.

The transaction history will automatically show any lightning related on-chain transactions (deposits/withdraws to your lightning wallet) as well as your Bitcoin Core wallet's on-chain transaction.

Any lightning related transaction will have a ⚡️ icon and any on-chain transactions will have a 🔗 icon, you will see funding and withdraw transactions with both a ⚡️ and 🔗icon because they are related to both your on-chain and lightning wallet. On-chain transactions are always denominated in btc, off-chain transactions always denominated in sats.

On the "send" view you will see some new lightning ⚡️ buttons.

Just above the address text input field there is a lightning ⚡️button, this will fetch a deposit address from your lightning node so you can easily deposit to it from your on-chain node's wallet.

There is a lightning ⚡️in the top right too, that is for withdrawing from your lightning node to whatever address you provide, amounts will be denominated in btc or USD depending on which currency you have selected (you can tap the coin button to toggle currency types).

In this view you can always paste in or scan a bolt11 invoice with an optional amount filled in, if the bolt11 invoice does not specify an amount you must fill one out in the amount input text field.

For creating channels go to "settings" > "node manager" > ⚡️ > ⚙️:
This will fetch all your peers and display their ID's. You can tap each one to see the raw json data from your c-lightning node.

You can tap the + button to scan a QR (that consists of <publicKey>@IP:port>) to connect to a peer, create and fund a channel.

First add an amount in satoshi's which you want to to commit to the channel.

Scanning a valid node url will trigger a series of calls:

- we establish a connection
- create a channel
- fund the channel (from your lightning wallet's on-chain wallet)
- confirm that the funding is secured

If all goes well you will get a success message, if not you'll get an error.

Thats it for now! It is a bit rough and ready but very functional, please report bugs, crashes and feature requests ⚡️⚡️⚡️⚡️⚡️⚡️⚡️