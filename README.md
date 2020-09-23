<div align="center">

## Sending complex emails in \.NET 1\.1


</div>

### Description

This article describes complex issues about sending emails in .NET 1.1 (such as using a SMTP server that requires authentication).
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Florentin BADEA](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/florentin-badea.md)
**Level**          |Intermediate
**User Rating**    |3.6 (29 globes from 8 users)
**Compatibility**  |C\#
**Category**       |[System Services/ Functions](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/system-services-functions__10-23.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/florentin-badea-sending-complex-emails-in-net-1-1__10-4550/archive/master.zip)





### Source Code

<H2>Introduction</H2>
<FONT size="2">
	<P>System.Web.Mail can be used to send emails from .NET 1.1 applications. Sending
		simple emails is very easy. More complicated is when you try to send emails
		using a SMTP server that requires authentication or even when you just need to
		format the From Name of the email you want to send.</P>
</FONT>
<H2>Background</H2>
<FONT size="2">
	<P>Here is the most simple piece of code that will send an email using C#:</P>
</FONT>
<PRE lang="cs">// build the email message
MailMessage&nbsp;msg = new MailMessage();
msg.From = "from.email@domain.com";
msg.To = "to.email@domain.com";
msg.Subject = "Subject";
msg.Body = "Body";
// send the message
SmtpMail.SmtpServer = "smtp.server.com";
SmtpMail.Send(msg);
&nbsp;
</PRE>
<H2>Problems and the immediate solution</H2>
<P>Things need to get more complicated if instead of displaying the from email
	address, you want to display a name that the recipient of the email will see.
	For that, a custom header needs to be added:</P>
<PRE lang="cs">string sFromName = "From display name";
string sFromAddress = "from.email@domain.com";
msg.Headers.Add("From", string.Format("{0} &lt;{1}&gt;", sFromName, sFromAddress));
</PRE>
<P>Even more complicated will be to send an email using a SMTP server that requires
	authentication. For that, the Fields collection of the MailMessage object needs
	to be used. Here is the sample piece of code that will help you solve your
	problems:</P>
<PRE lang="cs">// set SMTP server name
msg.Fields["http://schemas.microsoft.com/cdo/configuration/smtpserver"] = "smtp.server.com";
// set SMTP server port
msg.Fields["http://schemas.microsoft.com/cdo/configuration/smtpserverport"] = 25;
msg.Fields["http://schemas.microsoft.com/cdo/configuration/sendusing"] = 2;
msg.Fields["http://schemas.microsoft.com/cdo/configuration/smtpauthenticate"] = 1;
// set SMTP username
msg.Fields["http://schemas.microsoft.com/cdo/configuration/sendusername"] = "username";
// set SMTP user password
msg.Fields["http://schemas.microsoft.com/cdo/configuration/sendpassword"] = "password";
</PRE>
<H2>The better solution</H2>
<P>A better solution for the enhancements described above would be to create a new
	class that is inherited from MailMessage and has the extra features. Here is
	the content of the new class:
</P>
<PRE lang="cs">
/// <summary>
/// EnhancedMailMessage is a class that provides more features for email sending in .NET
/// </summary>
public class EnhancedMailMessage : MailMessage
{
	private string fromName;
	private string smtpServerName;
	private string smtpUserName;
	private string smtpUserPassword;
	private int smtpServerPort;
	public EnhancedMailMessage()
	{
		fromName = string.Empty;
		smtpServerName = string.Empty;
		smtpUserName = string.Empty;
		smtpUserPassword = string.Empty;
		smtpServerPort = 25;
	}
	/// <summary>
	/// The display name that will appear in the recipient mail client
	/// </summary>
	public string FromName
	{
		set
		{
			fromName = value;
		}
		get
		{
			return fromName;
		}
	}
	/// <summary>
	/// SMTP server (name or IP address)
	/// </summary>
	public string SMTPServerName
	{
		set
		{
			smtpServerName = value;
		}
		get
		{
			return smtpServerName;
		}
	}
	/// <summary>
	/// Username needed for a SMTP server that requires authentication
	/// </summary>
	public string SMTPUserName
	{
		set
		{
			smtpUserName = value;
		}
		get
		{
			return smtpUserName;
		}
	}
	/// <summary>
	/// Password needed for a SMTP server that requires authentication
	/// </summary>
	public string SMTPUserPassword
	{
		set
		{
			smtpUserPassword = value;
		}
		get
		{
			return smtpUserPassword;
		}
	}
	/// <summary>
	/// SMTP server port (default 25)
	/// </summary>
	public int SMTPServerPort
	{
		set
		{
			smtpServerPort = value;
		}
		get
		{
			return smtpServerPort;
		}
	}
	public void Send()
	{
		if (smtpServerName.Length == 0)
		{
			throw new Exception("SMTP Server not specified");
		}
		if (fromName.Length > 0)
		{
			this.Headers.Add("From", string.Format("{0} <{1}>", FromName, From));
		}
		// set SMTP server name
		this.Fields["http://schemas.microsoft.com/cdo/configuration/smtpserver"] = smtpServerName;
		// set SMTP server port
		this.Fields["http://schemas.microsoft.com/cdo/configuration/smtpserverport"] = smtpServerPort;
		this.Fields["http://schemas.microsoft.com/cdo/configuration/sendusing"] = 2;
		if (smtpUserName.Length >0 && smtpUserPassword.Length > 0)
		{
			this.Fields["http://schemas.microsoft.com/cdo/configuration/smtpauthenticate"] = 1;
			// set SMTP username
			this.Fields["http://schemas.microsoft.com/cdo/configuration/sendusername"] = smtpUserName;
			// set SMTP user password
			this.Fields["http://schemas.microsoft.com/cdo/configuration/sendpassword"] = smtpUserPassword;
		}
		SmtpMail.SmtpServer = smtpServerName;
		SmtpMail.Send(this);
	}
	public static void QuickSend(
		string SMTPServerName,
		string ToEmail,
		string FromEmail,
		string Subject,
		string Body,
		MailFormat BodyFormat)
	{
		EnhancedMailMessage msg = new EnhancedMailMessage();
		msg.From = FromEmail;
		msg.To = ToEmail;
		msg.Subject = Subject;
		msg.Body = Body;
		msg.BodyFormat = BodyFormat;
		msg.SMTPServerName = SMTPServerName;
		msg.Send();
	}
}
</PRE>
<P>As you can see from the code above you can send emails using SMTP servers that
	require authentication. Here is a sample usage code:</P>
<PRE lang="cs">EnhancedMailMessage msg = new EnhancedMailMessage();
msg.From = "from.email@domain.com";
msg.FromName = "From display name";
msg.To = "to.email@domain.com";
msg.Subject = "Subject";
msg.Body = "Body";
msg.SMTPServerName = "smtp.server.com";
msg.SMTPUserName = "username";
msg.SMTPUserPassword = "password";
msg.Send();
</PRE>
<P>Also, you can send emails using just on line of code:</P>
<PRE lang="cs">EnhancedMailMessage.QuickSend("smtp.server.com",
  "to.email@domain.com",
  "from.email@domain.com",
  "Subject",
  "Body",
  MailFormat.Html);
</PRE>
<H2>History
</H2>
<P>
27 Feb 2006 - first draft of the article<br>
28 Feb 2006 - created a wrapper class for all the code, created properties<br>
01 Mar 2006 - more comments in the class, performance and coding style improvements
</P>
<H2>The conclusion
</H2>
<P>Complex things can be done using simple .NET Framework classes. More things
	about email sending in one of the next articles.
</P>

