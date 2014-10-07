---
layout: post
title: "An In-depth Analysis of Linkedin's Intro"
date: 2013-10-26 11:51
comments: true
published: false
categories: 
---
<img src="{{root_url}}/images/headers/linkedin_intro.png"/>

### "Intro"duction
On October 23, Linkedin introduced an application called ["Intro"](http://blog.linkedin.com/2013/10/23/announcing-linkedin-intro/). The premise is simple: allow iPhone users to see details about the people they are emailing within the native iPhone Mail App. Think [Rapportive](http://rapportive.com/) for the iPhone Mail App, because that's *essentially* what this is (and made by the same people).

However, to accomplish this feat, the team at Linkedin had to resort to some clever engineering and questionable choices. While there have been other analyses published into the impact of using Intro, I have yet to see an in-depth technical look into how it all actually works behind the scenes from a user's perspective. Hopefully, this post will shed some light and show what risks users are actually taking when using the service.
<!-- more -->

### Installing Intro
Linkedin made installing Intro pretty simple
https://secure.intro.linkedin.com/installations/iphone
To register -
/installations/auth?email_address=jmwright798%40gmail.com
Oauth:
/o/oauth2/auth?access_type=offline&approval_prompt=force&client_id=357812332413.apps.googleusercontent.com&login_hint=jmwright798%40gmail.com&redirect_uri=https%3A%2F%2Fsecure.intro.linkedin.com%2Finstallations%2Foauth2callback&response_type=code&scope=https%3A%2F%2Fmail.google.com%2F+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+profile&state=csrf_token%3Dad0f63a1f31ddddea6d86b333e38bb16%26login_hint%3Djmwright798%2540gmail.com

Profile:
/installations/[md5]/device_info.mobileconfig"

/rapportive-ca/ejbca/publicweb/apply/scep/pkiclient.exe?operation=GetCACert&message=EnrollmentCAInstance

/rapportive-ca/ejbca/publicweb/apply/scep/pkiclient.exe?operation=PKIOperation&message=MIAGCSqGSIb3DQEHAqCAMIACAQExCzAJBgUrDgMCGgUAMIAGCSqGSIb3DQEHAaCAJIAEggR1MIAGCSqGSIb3DQEHA6CAMIACAQAxgdQwgdECAQAwOjAuMSwwKgYDVQQDDCNSYXBwb3J0aXZlIEVucm9sbG1lbnQgQ0EgMjAxMy0wMy0wOAIIK9S6EZCzCMAwDQYJKoZIhvcNAQEBBQAEgYCGzl35AXdzralIRcUrf2y9yWPjE3aKBUXJacL25HnE5UAyeqsGIDrzxIIx3GTRb0rv0S0KGKHbkWR6%2FDuYCOb2hBIWwVB6KXXuN9sAsJhmBDjwXjY2OvCjpxdcpGE2bdG3IiEs0ORktOq2mPDvgst1Z49Qm%2BfjtEG1Di9P2TJabjCABgkqhkiG9w0BBwEwEQYFKw4DAgcECJFVPTis7XWcoIAEggNQZHMLOsLphX56fsutDXtzYtvxDoLsU0s3RNC2OevMlKj39swa9Duj7174%2FuFSKBiO1wSFLj09jxlxmiIFHTkoK2aGkVULeKmp%2FyxCKQLWuLoBzo1AN0302dXqn3X6etdqzW6qmoY%2FiKnUBxLE5HbDbLy7%2F2aGXJMlsX3ZzmHG%2Fos2uJOeENp9KLyfM5UyxXxC8i1f4WPnI2ux1t44%2BjnPB79PkeKG9wok4KyyWjt47ts%2FMfFZox44J7%2B7jOZAwFhgXkM9c1%2FJbVHh60Saj%2BqhzZueppgLyMPUmMlKykszyjRU8uPJJ2dBf2nf4K0eOScwjMR5LzB%2BuwyQw65chudzKVXftHKeg2i%2BygKW%2BKi8NdneEiF3EMeYmf7CBGov%2FnlT1klOCBg210qF10ZeZ9Y6kjRCVIa8nfMwK%2Bk%2Fz1nsSuMgL3FZWgyXxxxTs9jqcvzrt0auB4Hhvx85Gw2rPOVVy2A4btac0nI5MiUW3%2BwpoqZxzPpn8OovA7OuU3SGUd3ooJFzxAKlsd3Rm2CBKmWt7qP9JoJrXdKI%2FA97FJylS8GtFAGOY2MTMxpfpQD5aop8L9J9ZBVUJ7IdEZle9HFpfnqm2gXOeZLFJJ7HknR1qNyhqVypjNi3KYlKwWuybM0X0PL6BJELaAvogt%2BXIeG9dNPliOOVW%2B%2F%2BVtgermHrQumWo3J8kt7iRHmowGSzP0I16%2BGYMkaYFZwT%2F7C9IjpCPhV0B2j683l%2FYmpMMn%2FFkI1wB3Y6%2B3HvwUxLB2yaAJQQxu5u%2B4NIEfneZ0n7EuTHziChJDPJIRoURfIHwUh0mPwVze8Jb%2BwSDyPh5BTd93W77Xbq22%2BC%2FT17QKV%2Fs3FeFMEzNk3qvFH9zWc2be3mEdHVmnFRzp98gExwQNTfBYkUWWyqA%2BqW1RHLWswbLg%2Bih%2FTXatrfSJVz%2FkyPsLMUSgmNPlf11PQttuAoTWq7CMDZgSm%2BORqAf5%2FOIk%2Bw68TlL6Vd8kOmMDMr0UhYeetTfziuzPSi115AFdzUvkF1SFeNWoLJofSnG6MIew480Elt8%2FqPgfgWzq0hJ5Y0CjfsbeW5apXoDv6b86bUKLohaYL0cRUNQweOnGDCyWHXus2PvTA%2FyRUj3ViygkStesNj9gEECPtMSN8z4H7mAAAAAAAAAAAAAAAAAAAAAKCCAeowggHmMIIBT6ADAgECAgEBMA0GCSqGSIb3DQEBBQUAMC8xLTArBgNVBAMTJENDQTcyRDlDLUUxMEEtNDg0Ni05NDhCLTcxNkFFNDYxQkM0NTAeFw0xMzEwMjYyMTI4MjVaFw0xNDEwMjYyMTI4MjVaMC8xLTArBgNVBAMTJENDQTcyRDlDLUUxMEEtNDg0Ni05NDhCLTcxNkFFNDYxQkM0NTCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEAn3R8adVJyVzWlqACn1sex0%2F0ll4jPC8a0z9usRFUTfbPn83a7HTUaepzUqrVZ%2BjoZfO8FY2Fu5tH7XyZzcXf7u9V3elkHfNwfEGmF%2BUVNWjUJY3J3wOqwCKsY0Zkee669zzFxsMSP6xW3MMV4iRLKxbB3fh1WnBaqg7oWIR4ECUCAwEAAaMSMBAwDgYDVR0PAQH%2FBAQDAgWgMA0GCSqGSIb3DQEBBQUAA4GBAJoYRCagDTSsb7dTXrFlawCe6QcR3nTnK%2Fx%2FRdfLnG%2Bn7l6CvePev8yGDqaJhyM1tnXPn1y4%2BU2zmdsGL%2FiZK9cT%2BsYoUevGDIZs7Snq8kP9fYTEs8NHHFni2eArDeumU%2BzBPz9mDElAp4f%2BptF2D2ONyS8l%2B194L6%2FYF2ZpF9PgMYIBqjCCAaYCAQEwNDAvMS0wKwYDVQQDEyRDQ0E3MkQ5Qy1FMTBBLTQ4NDYtOTQ4Qi03MTZBRTQ2MUJDNDUCAQEwCQYFKw4DAhoFAKCBzTASBgpghkgBhvhFAQkCMQQTAjE5MBgGCSqGSIb3DQEJAzELBgkqhkiG9w0BBwEwHAYJKoZIhvcNAQkFMQ8XDTEzMTAyNjIxMjgyNVowIAYKYIZIAYb4RQEJBTESBBDUfdvukbp9YZP%2BRfJzNiroMCMGCSqGSIb3DQEJBDEWBBSybaduyomCd8gYAnHj%2FdCvpHCa%2FDA4BgpghkgBhvhFAQkHMSoTKDAwNkM4NkZDMkU3NDY3MkM2MThDQzU3MjBCRjQyMkJFREVEQ0REQjYwDQYJKoZIhvcNAQEBBQAEgYA%2BWHu%2FKVWBbHQMKjKg5Vlug4jMz79S3xbRGJ36NJuU3thtsenXkNZmOS7F2%2FRGYBKr7O%2B7Fjsv0x8Vw8MxHGHZafJSl7WV7dE681trV1%2BLaO%2FsJu94OaSs7D5QnqCxAPpj%2BsyLO6yjI71dEsuxULJiUMddDnhiFTTOCdvmF2LhYAAAAAAAAA%3D%3D





### What Happens to Your Mail?

quiet.intro.linkedin.com/mail/linkedin/status?checksum=fa51137ad85023fee3d39e91fdb0aaa03014068f1a4c9e6a6d3eeddcc7415066&default=not_connected&device_type=iphone&email=michael.bazzell%40gmail.com&user_id=56476

/via?a=AgwC3MuMuAEABAKSrBACv7VX82sVOkgS4Y3QrXVS6wIxpnkA9lR8Tgh-8vo95hPKAgIKNi4xLjMCBAAAAAAABAQEBgAAAB8_ZEuEjIGkC1EFVHQBtCFTmQnrmN7csAyIyWBXmI0E&redirect=

Downloads profile:
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>PayloadType</key><string>Configuration</string>
  <key>PayloadUUID</key><string>0080501b-828b-48e3-a767-eeb66057e835</string>
  <key>PayloadIdentifier</key><string>com.rapportive.ios.configuration.1db861e2d444760f09a0efa684dfb1b5</string>
  <key>PayloadDescription</key><string>Tap &apos;Done&apos; to continue.


Â </string>
  <key>PayloadContent</key><array>
  <dict>
    <key>PayloadDisplayName</key><string>Intro</string>
    <key>PayloadDescription</key><string>Manage your LinkedIn Intro account</string>
    <key>PayloadType</key><string>com.apple.webClip.managed</string>
    <key>PayloadVersion</key><integer>1</integer>
    <key>PayloadIdentifier</key><string>com.rapportive.ios.webclip</string>
    <key>PayloadUUID</key><string>6dc8ccd8-3dd1-4729-a8af-39d6c3567fe6</string>
    <key>Label</key><string>Intro</string>
    <key>URL</key><string>https://secure.intro.linkedin.com/mobile/iphone</string>
    <key>Icon</key><data>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
AAAAAElFTkSuQmCC
</data>
    <key>Precomposed</key><true/>
    <key>IsRemovable</key><false/>
    <key>FullScreen</key><false/>
  </dict>
  <dict>
    <key>Password</key><string>c239affa5103e273423c6e087ab8ba4f</string>
    <key>PayloadCertificateFileName</key><string>c1f6e5c96c9ccabb4b8e5e70ad47d761.p12</string>
    <key>PayloadContent</key><data>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
4o2FnH1ZSFZD2HwZBAips7N1RsWLKw==
</data>
    <key>PayloadDescription</key><string>Provides device authentication (certificate or identity).</string>
    <key>PayloadDisplayName</key><string>Intro</string>
    <key>PayloadIdentifier</key><string>com.rapportive.pkcs12.c1f6e5c96c9ccabb4b8e5e70ad47d761</string>
    <key>PayloadOrganization</key><string></string>
    <key>PayloadType</key><string>com.apple.security.pkcs12</string>
    <key>PayloadUUID</key><string>c5ba9e18-42da-4a98-8c36-ef531b52122b</string>
    <key>PayloadVersion</key><integer>1</integer>
  </dict>
  <dict>
    <key>PayloadDisplayName</key><string>Email Settings</string>
    <key>PayloadType</key><string>com.apple.mail.managed</string>
    <key>PayloadVersion</key><integer>1</integer>
    <key>PayloadUUID</key><string>66a68471-513e-459f-85f0-150dcc1df46e</string>
    <key>PayloadIdentifier</key><string>com.rapportive.iphone.settings.email.1db861e2d444760f09a0efa684dfb1b5</string>
    <key>EmailAccountName</key><string>Test Account</string>
    <key>EmailAccountType</key><string>EmailTypeIMAP</string>
    <key>EmailAddress</key><string>linkedin.intro.test@gmail.com</string>
    <key>EmailAccountDescription</key><string>Gmail +Intro</string>
    <key>IncomingMailServerAuthentication</key><string>EmailAuthPassword</string>
    <key>IncomingMailServerHostName</key><string>imap.intro.linkedin.com</string>
    <key>IncomingMailServerPortNumber</key><integer>143</integer>
    <key>IncomingMailServerUseSSL</key><true/>
    <key>IncomingMailServerUsername</key><string>AgY6bGlua2VkaW4uaW50cm8udGVzdEBnbWFpbC5jb23B9uXJbJzKu0uOXnCtR9dhANj3BrS1EABaMS9kMWZxQUZZajE4Y0hSc1FZdGJFc2k3Zy1lblhFNi1paFIyeUhENmVSbk1BFwx8vMNyGnPn18EjErKCKadxuO0zAPxLw4j-gZu7uUw=</string>
    <key>IncomingPassword</key><string>8f2a913689ce39547a718e5965a06d9b</string>
    <key>OutgoingPasswordSameAsIncomingPassword</key><true/>
    <key>OutgoingMailServerAuthentication</key><string>EmailAuthPassword</string>
    <key>OutgoingMailServerHostName</key><string>smtp.intro.linkedin.com</string>
    <key>OutgoingMailServerPortNumber</key><integer>587</integer>
    <key>OutgoingMailServerUseSSL</key><true/>
    <key>OutgoingMailServerUsername</key><string> Gmail +Intro ?Agg6bGlua2VkaW4uaW50cm8udGVzdEBnbWFpbC5jb206bGlua2VkaW4uaW50cm8udGVzdEBnbWFpbC5jb23B9uXJbJzKu0uOXnCtR9dh2PcGAABaMS9kMWZxQUZZajE4Y0hSc1FZdGJFc2k3Zy1lblhFNi1paFIyeUhENmVSbk1BxS5sUuRy7tlxiZ7b-rPGMlkVhQMemshXkc6q3g7z6wc=</string>
    <key>OutgoingPassword</key><string>8f2a913689ce39547a718e5965a06d9b</string>
  </dict>
</array>
  <key>PayloadDisplayName</key><string>Intro (linkedin.intro.test@gmail.com)</string>
  <key>PayloadVersion</key><integer>1</integer>
  <key>PayloadOrganization</key><string>LinkedIn</string>
</dict>
</plist>


### Possible Attack Vectors

### Conclusion

AgY6bGlua2VkaW4uaW50cm8udGVzdEBnbWFpbC5jb23B9uXJbJzKu0uOXnCtR9dhANj3BrS1EABaMS9kMWZxQUZZajE4Y0hSc1FZdGJFc2k3Zy1lblhFNi1paFIyeUhENmVSbk1BFwx8vMNyGnPn18EjErKCKadxuO0zAPxLw4j-gZu7uUw= 862cb72b3430baaf63a6fa3a844f6b61
