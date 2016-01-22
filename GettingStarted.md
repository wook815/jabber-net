# What Is Jabber-Net? #

Jabber-Net is a set of libraries for accessing Jabber functionality from .Net. It is written in C#, but should be accessible from other .Net languages such as VB.Net. Classes exist for connecting to a Jabber server either as a client or as a component. As you explore, you'll find there are some other goodies buried inside, like Trees, CommandLine processing, etc.


# How Do I Use Jabber-Net? #

  * Create a new windows app in VS.Net.
  * Right-click in toolbox, select "Add Tab".  Name the tab "Jabber-Net"
  * In the new tab, right-click and select "Choose Items...".
  * Select .the .Net Framework Components tab
  * Click browse, and select jabber-net.dll, prefereably in the bin/Debug directory, where it gets built
  * Do the same for muzzle.dll (UI elements) if you want
  * Click OK, to add a the components
  * Drop a JabberClient component on your form
  * Set connection parameters in the property box
  * Call jabberClient1.Connect() in the Form.OnLoad event handler
  * Go to events in the property box (the lightning bolt), double-click OnMessage
  * Write code to handle Message's, like this:

```
private void jabberClient1_OnMessage(object sender, Message msg)
{
  jabber.protocol.client.Message reply = new jabber.protocol.client.Message(jabberClient1.Document);			
  reply.Body = "Hello!";
  reply.To = msg.From;
  jabberClient1.Write(reply);
}
```

Note that packet types such as Message are sub-classes of XmlElement with easy-to-use getters and setters.