(thanks to Tom Waters for the genesis for this)

Say you want to use a new packet type like this:
```
<!-- get your own list of all your objects... -->
<iq type='get' to='self' id='n0'>
 <query xmlns='your:namespace'/>
</iq>

<!-- reply with list of objects... -->
<iq type='result' to='self' id='n0'>
 <query xmlns='your:namespace'>
  <yourobj key='Object1' other='value1'/>
 </query>
</iq>
```

First, make sure you implement two constructors for each of your types:

```
namespace your.protocol
{
    public class YourQuery : Element 
    {
        public const string YOUR_NS  = "your:namespace";

        // used when creating elements to send
        public YourQuery(XmlDocument doc) : base("query", YOUR_NS, doc)  
        {}

        // used to create elements for inbound protocol
        public YourQuery(string prefix, XmlQualifiedName qname, XmlDocument doc)
            : base(prefix, qname, doc) 
        {}

        // put your accessor methods here
    }
}
```

In order to get the inbound connection to create objects of your class, rather than just plain `XmlElement`s, you need to create a factory class that maps the combination of the element name and namespace to a type.

```
public class Factory : jabber.protocol.IPacketTypes 
{
    private static QnameType[] s_qnt = new QnameType[] 
    {
        new QnameType("query", YourQuery.YOUR_NS, typeof(your.protocol.YourQuery))
        // Add other types here, perhaps sub-elements of query...
    };
    QnameType[] IPacketTypes.Types { get { return s_qnt; } }
}
```


Next, register your factory with your connection object in the `OnStreamInit` event.

```
private void jabberClient_OnStreamInit(object sender, ElementStream stream)
{
    stream.AddFactory(new your.protocol.Factory());
}
```


One more note.  Most of the classes you are liable to write should derive from jabber.protocol.Element.  jabber.protocol.Packet is for top-level jabber stanzas, like `<message/>`, `<iq/>`, and `<presence/>`.