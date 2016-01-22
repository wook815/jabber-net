For background, see [XEP-0077](http://www.xmpp.org/extensions/xep-0077.html).

Do something like this:

```
// Alternately, you can listen for OnAuthError, and try to register at that point.
jc.AutoLogin = false;
jc.OnLoginRequired += new bedrock.ObjectHandler(jc_OnLoginRequired);

jc.OnRegisterInfo += new RegisterInfoHandler(this.jc_OnRegisterInfo);
jc.OnRegistered += new IQHandler(jc_OnRegistered);

void jc_OnLoginRequired(object sender)
{
    jc.Register(new JID(jc.User, jc.Server, null));
}

private bool jc_OnRegisterInfo(object sender, Register r)
{
    // Fill in r with whatever data you can.  See XEP-77.

    // If there is no x:data form, we could fill in the old-school parameters;
    // return true to take the defaults.
    if (r.Form == null)
        return true;

    // Modern servers will send an x:data form.
     muzzle.XDataForm f = new muzzle.XDataForm(r.Form);
     if (f.ShowDialog() != DialogResult.OK)
          return false;  // Cancel the registration process.

     f.FillInResponse(r.Form);
     return true;
}

private void jc_OnRegistered(object sender, IQ iq)
{
    // If the registration worked, let's try to authenticate.
    if (iq.Type == IQType.result)
        jc.Login();
}


```