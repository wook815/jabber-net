# Introduction #

Jabber-Net uses an asynchronous programming model by default.  This can be confusing if you are new to .Net.  This example shows how to set up a connection, connect, and send a message before your program finishes.  The key concept is the use of [ManualResetEvent](http://msdn2.microsoft.com/en-us/library/system.threading.manualresetevent.aspx) to wait for the end of asynchronous processing.


# SendMessage.cs #

Make sure to reference `jabber-net.dll`, `zlib.net.dll`, and `netlib.Dns.dll`.

```
using System;
using System.Threading;

using jabber.client;

namespace SendMessage
{
    class Program
    {
        // we will wait on this event until we're done sending
        static ManualResetEvent done = new ManualResetEvent(false);
        // if true, output protocol trace to stdout
        const bool VERBOSE = true;
        const string TARGET = "otheruser@example.com";

        static void Main(string[] args)
        {
            JabberClient j = new JabberClient();
            // what user/pass to log in as
            j.User = "someuser";
            j.Server = "example.com";  // use gmail.com for GoogleTalk
            j.Password = "somepassword";

            // don't do extra stuff, please.
            j.AutoPresence = false;
            j.AutoRoster = false;
            j.AutoReconnect = -1;

            // listen for errors.  Always do this!
            j.OnError += new bedrock.ExceptionHandler(j_OnError);

            // what to do when login completes
            j.OnAuthenticate += new bedrock.ObjectHandler(j_OnAuthenticate);

            // listen for XMPP wire protocol
            if (VERBOSE)
            {
                j.OnReadText += new bedrock.TextHandler(j_OnReadText);
                j.OnWriteText += new bedrock.TextHandler(j_OnWriteText);
            }
            
            // Set everything in motion
            j.Connect();

            // wait until sending a message is complete
            done.WaitOne();

            // logout cleanly
            j.Close();
        }

        static void j_OnWriteText(object sender, string txt)
        {
            if (txt == " ") return;  // ignore keep-alive spaces
            Console.WriteLine("SEND: " + txt);
        }

        static void j_OnReadText(object sender, string txt)
        {
            if (txt == " ") return;  // ignore keep-alive spaces
            Console.WriteLine("RECV: " + txt);
        }

        static void j_OnAuthenticate(object sender)
        {
            // Sender is always the JabberClient.
            JabberClient j = (JabberClient)sender;
            j.Message(TARGET, "test");

            // Finished sending.  Shut down.
            done.Set();
        }

        static void j_OnError(object sender, Exception ex)
        {
            // There was an error!
            Console.WriteLine("Error: " + ex.ToString());

            // Shut down.
            done.Set();
        }
    }
}
```