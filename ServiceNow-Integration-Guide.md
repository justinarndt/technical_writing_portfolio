# **Integration Guide: Automated P1 Incident Alerts (ServiceNow to Slack)**

Version: 2.1  
Target Audience: ITSM Administrators & DevOps Leads  
Prerequisites: ServiceNow "Admin" role, Slack Webhook URL

## **1\. Overview**

This integration reduces **Mean Time to Response (MTTR)** by automatically pushing Priority 1 (Critical) incidents from ServiceNow directly to a dedicated Slack channel (\#ops-critical). This removes the delay caused by email notifications and ensures immediate visibility for Site Reliability Engineers (SREs).

### **Workflow Logic**

When an Incident record is created or updated:

1. **Business Rule** triggers if Priority is 1 \- Critical.  
2. **REST Message** constructs a JSON payload.  
3. **Slack API** receives the payload and posts a formatted alert.

sequenceDiagram  
    participant User as End User  
    participant SN as ServiceNow (Business Rule)  
    participant Slack as Slack Webhook  
    participant SRE as DevOps Team

    User-\>\>SN: Reports Critical Issue (P1)  
    SN-\>\>SN: Check Condition (Priority \== 1\)  
    SN-\>\>Slack: POST /services/hooks/incoming-webhook  
    Slack-\>\>SRE: 🚨 New P1 Incident: Database Down

## **2\. Configuration Steps**

### **Step 1: Create the Outbound REST Message**

1. Navigate to **System Web Services \> Outbound \> REST Message**.  
2. Click **New**.  
   * **Name:** Slack\_Alert\_Integration  
   * **Endpoint:** https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX  
3. Under **HTTP Methods**, open the default post method.  
4. Set **HTTP Content** to the following JSON structure:

{  
    "text": "🚨 \*Critical Incident Reported\*",  
    "attachments": \[  
        {  
            "color": "\#ff0000",  
            "fields": \[  
                { "title": "Incident", "value": "${number}", "short": true },  
                { "title": "Caller", "value": "${caller\_id}", "short": true },  
                { "title": "Description", "value": "${short\_description}", "short": false }  
            \]  
        }  
    \]  
}

### **Step 2: Configure the Business Rule**

This script executes the logic. ServiceNow uses **Rhino JavaScript**.

1. Navigate to **System Definition \> Business Rules**.  
2. Click **New**.  
   * **Name:** Trigger P1 Slack Alert  
   * **Table:** Incident \[incident\]  
   * **When:** After (Insert/Update)  
3. **Filter Conditions:** Priority **is** 1 \- Critical.  
4. **Advanced Tab (Script):**

(function executeRule(current, previous /\*null when async\*/) {

    try {  
        var r \= new sn\_ws.RESTMessageV2('Slack\_Alert\_Integration', 'post');  
          
        // Map dynamic fields from the Incident record to the JSON payload  
        r.setStringParameterNoEscape('number', current.number);  
        r.setStringParameterNoEscape('caller\_id', current.caller\_id.getDisplayValue());  
        r.setStringParameterNoEscape('short\_description', current.short\_description);

        var response \= r.execute();  
        var responseBody \= response.getBody();  
        var httpStatus \= response.getStatusCode();  
          
        gs.info("Slack Alert Sent: " \+ httpStatus); // Log for audit trails

    } catch (ex) {  
        var message \= ex.message;  
        gs.error("Slack Integration Failed: " \+ message);  
    }

})(current, previous);

## **3\. Validation & Troubleshooting**

To verify the integration, use the **Script Background** module to simulate a P1 incident without emailing the entire company.

**Test Script:**

var inc \= new GlideRecord('incident');  
inc.initialize();  
inc.short\_description \= "TEST \- Database Connectivity Loss";  
inc.urgency \= 1; // High  
inc.impact \= 1;  // High  
// Priority is calculated automatically based on Urgency \+ Impact  
inc.insert();

**Troubleshooting Table:**

| HTTP Code | Meaning | Resolution |
| :---- | :---- | :---- |
| **200 OK** | Success | Message should appear in Slack immediately. |
| **403 Forbidden** | Invalid Token | Check if the Webhook URL was rotated in Slack. |
| **500 Error** | Server Error | Check System Logs for script syntax errors. |

