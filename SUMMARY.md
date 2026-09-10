# Table of Contents

* [Home](README.md)

## Getting Started

* [Web Application Pentesting Checklist](CheckList/Web-Application-Pentesting-checklist.md)
* [Recon Workflow](Recon/Workflow.md)
* [Subdomain Enumeration](Recon/subdomain_enumeration.md)
* [GitHub Dorking](Recon/Github_Dorking.md)
* [Web Source Code Review Tips](Web_Source_Review/codereviewtips.md)

## Account Takeover & Authentication

* [Account Takeover Methodologies](Account_Takeovers_Methodologies/Account_Takeovers_Methods.md)
* [Account Ban Bypass](Authentication_Bypass/account_ban_bypass.md)
* [OTP Bypass](Authentication_Bypass/OTP_Bypass.md)
* [2FA/MFA Bypasses](MFA_Bypasses/2FA_Bypass.md)
* [Session-Based Bugs](Broken_Auth_And_Session_Management/Session_based_bugs.md)
* [Sign-Up Functionality Bugs](Sign_Up_Functionality/Hunting_for_bugs_in_signup_feature.md)
* [Password Reset Functionality](Password_Reset_Functionality/README.md)
  * [Top 5 Password Reset Bugs](Password_Reset_Functionality/Top_5_Password_Reset_Bugs.md)
  * [Account Takeover via Password Reset](Password_Reset_Functionality/Account_Takeover_By_Password_Reset_Functionality.md)
  * [Password Reset Flaws](Password_Reset_Functionality/Password_Reset_Flaws_by_Sm4rty.md)
  * [Password Reset Token Leakage](Password_Reset_Functionality/Password_Reset_Token_Leakage.md)
  * [HTML Injection on Password Reset Page](HTML_Injection/HTML_Injection_on_password_reset_page.md)
* [Weak Password Policy](Weak_Password_Policy/Weak_password_policy.md)

## Identity & Tokens

* [JWT Attacks](JWT/JWT.md)
* [OAuth 2.0 Hunting Methodology](OAuth/OAuth_2.0_Hunting_Methodology.md)
* [SAML Hunting](SAML/SAML.md)

## Client-Side

* [XSS](XSS/Xss.md)
  * [XSS Bypasses](XSS/XSS_Bypass.md)
  * [postMessage XSS](XSS/post_message_xss.md)
  * [CSP Bypasses](XSS/Bypass_CSP.md)
  * [Automated XSS](XSS/Automated_XSS.md)
* [CORS Misconfiguration](CORS/CORS.md)
  * [CORS Bypasses](CORS/CORS_Bypasses.md)
* [CSRF](CSRF/CSRF.md)
  * [CSRF Bypasses](CSRF/Cross_Site_Request_Forgery_Bypass.md)
* [Open Redirection Hunting](Open_Redirection/Open_Redirection_Hunting.md)
  * [Open Redirection Bypasses](Open_Redirection/Open_Redirection_Bypass.md)
* [Tabnabbing](Tabnabbing/Tabnabbing.md)
* [Host Header Attacks](Host-Header/Host-Header.md)
* [Parameter Pollution in Social Sharing Buttons](Parameter_Pollution/Parameter_Pollution_in_social_sharing_buttons.md)

## Server-Side

* [IDOR](IDOR/IDOR.md)
* [SQL Injection](SQLi/SQL_Injection.md)
* [SSRF](SSRF/SSRF.md)
  * [Blind SSRF](SSRF/Blind_SSRF.md)
* [SSTI](SSTI/SSTI.md)
* [XXE](XXE/XXE_Methods.md)
  * [Billion Laughs Attack](XXE/Billion_Laugh_Attack.md)
* [File Upload Vulnerabilities](File_Upload/file_upload.md)
* [Race Conditions](Race_Condition/race_conditions.md)
* [Application Level DoS](Application_Level_DoS/ALD_Methods.md)
* [HTTP Desync](HTTP_Desync/http_desync.md)
* [Status Code / 403 Bypasses](Status_Code_Bypass/403Bypass.md)
* [WAF Bypass Using Headers](WAF_Bypasses/WAF_Bypass_Using_headers.md)

## Rate Limiting

* [Rate Limit Guide (Index)](Rate_limit/README.md)
* [Missing Rate Limit Scenarios](Rate_limit/Missing_Rate_Limit_Scenarios.md)
* [Rate Limit Bypasses](Rate_limit/RateLimitBypass.md)

## APIs & GraphQL

* [API Testing (Index)](API_Testing/README.md)
* [Hidden API Functionality Exposure](API_Testing/Hidden_API_Functionality_Exposure.md)
* [Reverse Engineer an API](API_Testing/Reverse_Engineer_an_API.md)
* [GraphQL Hunting](GraphQL/GraphQL.md)

## Misconfigurations & Leaks

* [Sensitive Info Leaks (Index)](Sensitive_Info_Leaks/README.md)
  * [GitHub Dorks (Curated)](Sensitive_Info_Leaks/Github-dorks.md)
  * [GitHub Dorks (Full List)](Sensitive_Info_Leaks/Github_dorks_all.md)
  * [GitHub Recon Method](Sensitive_Info_Leaks/Github_Recon_Method.md)
  * [Google Dorks](Sensitive_Info_Leaks/Google_Dorks.md)
  * [Shodan CVE Dorks](Sensitive_Info_Leaks/Shodan_cve_dorks.md)
  * [Version Leaks](Sensitive_Info_Leaks/Version_Leak.md)
* [Default Credentials & Admin Panels](Misconfigurations/Default_Credential_And_Admin_Panel.md)
* [Docker Misconfigurations](Misconfigurations/Docker.md)
* [S3 Bucket Misconfigurations](Misconfigurations/S3-Bucket_Misconfig.md)
* [EXIF Geo Data Not Stripped](EXIF_Geo_Data_Not_Stripped/exif_geo.md)
* [Finding Origin IP](FindOriginIP/FindOrigin.md)
* [Broken-Link Hijacking](BrokenLinkHijacking/BrokenLinkHijacking.md)

## Subdomain Takeover

* [Subdomain Takeover Guide](Subdomain_Takeover/Subdomain_Takeover.md)
* [Easy Methods](Subdomain_Takeover/Easy_Methods.md)
* [Sub / Top-Level Domain Takeover](Subdomain_Takeover/Sub_or_top_level_domain_takeover.md)

## CMS-Specific

* [WordPress](CMS/wordpress.md)
* [Drupal](CMS/Drupal.md)
* [Moodle](CMS/Moodle.md)
* [Adobe AEM](CMS/AEM.md)

## Target-Specific & CVE Hunting

* [Jira Hunting](JIRA/README.md)
* [Easy CVE Hunting](CVES/easycve.md)
