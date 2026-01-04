# Autonomous RFP Analyzer  n8n Workflow

## 📋 Overview

This n8n workflow automatically processes incoming RFP (Request for Proposal) documents, analyzes them using AI, and routes them to the appropriate destination based on fit assessment. It includes a human-in-the-loop approval mechanism for quality control.

## 🎯 Purpose

**Business Problem:** Trilles AI receives RFPs as PDF attachments that need quick evaluation to determine if they're worth pursuing.

**Solution:** An automated pipeline that:
- Monitors for new RFP documents
- Extracts and analyzes content using AI
- Determines project fit based on company competencies (n8n/Python)
- Routes qualified opportunities for human approval
- Archives rejected proposals with reasoning

## 🔄 Workflow Architecture

### Main Flow Sequence

```
Manual Trigger → Google Drive → Extract PDF → AI Agent → Code Parser → Decision Branch
                                                                              ↓
                                                                    ┌─────────┴─────────┐
                                                                    ↓                   ↓
                                                            GOOD FIT                BAD FIT
                                                                    ↓                   ↓
                                                          Gmail Alert           Archive to
                                                                ↓               Baserow Table
                                                            Wait for
                                                            Response
                                                                ↓
                                                          If Approved
                                                                ↓
                                                          Save to
                                                    Opportunities Table
```

## 🔧 Node-by-Node Breakdown

### 1. **Manual Trigger** (Execute Workflow)
- **Purpose:** Initiates the workflow on demand
- **Type:** Manual execution
- **Use Case:** Testing or manual processing of specific RFPs

### 2. **Google Drive**
- **Purpose:** Retrieves RFP documents from Google Drive
- **Configuration:**
  - Monitors a specific folder (e.g., "RFP_Inbox")
  - Downloads PDF files automatically
- **Output:** Binary PDF file data

### 3. **Extract from File**
- **Purpose:** Converts PDF to readable text
- **Method:** Extracts text content from PDF binary
- **Output:** Plain text string of document content

### 4. **AI Agent** (Google Gemini Chat Model)
- **Purpose:** Intelligent analysis of RFP content
- **AI Model:** Google Gemini
- **Analysis Tasks:**
  - Extract Budget
  - Identify Deadline
  - List Tech Stack requirements
  - Assess fit with company competencies (n8n/Python)
  - Generate confidence score
  - Create brief summary
  - Provide rejection reasoning (if applicable)

**Prompt Structure:**
```
Analyze this RFP and extract:
1. Budget (number or "unknown")
2. Deadline (date format)
3. Tech Stack (list of technologies)
4. Good fit for n8n/Python automation? (yes/no)
5. Confidence score (0-100%)
6. Summary (2-3 sentences)
7. Rejection reason (if not a fit)

Return as JSON format.
```

### 5. **Code in JavaScript**
- **Purpose:** Parse and structure AI response
- **Operations:**
  - Clean JSON formatting
  - Extract key fields
  - Structure data for downstream nodes
- **Output:** Structured JSON object with analysis results

### 6. **IF (Conditional Branch)**
- **Purpose:** Route workflow based on fit assessment
- **Condition:** `good_fit === true`
- **Branches:**
  - **TRUE:** Proceeds to approval workflow
  - **FALSE:** Archives as rejected opportunity

---

## 📧 Approval Path (TRUE Branch)

### 7. **Gmail - Send Message**
- **Purpose:** Notify decision-maker via email
- **Recipients:** Sales team/Project manager
- **Content Includes:**
  - RFP name and source
  - Budget and deadline
  - Tech stack requirements
  - AI confidence score
  - Executive summary
  - Link or instructions for approval

### 8. **Wait (Pause Node)**
- **Purpose:** Human-in-the-loop approval gate
- **Behavior:** Workflow pauses until response received
- **Wait Trigger:** Email reply, webhook, or form submission
- **Timeout:** Configurable (e.g., 48 hours)

### 9. **IF1 (Second Conditional)**
- **Purpose:** Check approval decision
- **Condition:** Approval confirmed
- **Branches:**
  - **Approved:** Save to Opportunities
  - **Rejected:** Route to Archive

### 10. **Baserow - Create Row (Opportunities)**
- **Purpose:** Store approved RFPs
- **Table:** "Opportunities"
- **Fields Saved:**
  - RFP Name
  - Budget
  - Deadline
  - Tech Stack
  - Status: "Pending Review"
  - Summary
  - Confidence Score
  - Date Added

---

## 🗄️ Archive Path (FALSE Branch)

### 11. **Baserow - Create Row1 (Archive)**
- **Purpose:** Log rejected RFPs
- **Table:** "Archive"
- **Fields Saved:**
  - RFP Name
  - Rejection Reason
  - AI Confidence Score
  - Date Processed
  - Tech Stack (for reference)

---

## ⚙️ Setup Requirements

### Prerequisites

1. **n8n Instance** (cloud or self-hosted)
2. **Google Drive Account** with API access
3. **Gmail Account** for notifications
4. **Baserow Account** with two tables:
   - **Opportunities Table** columns:
     - RFP_Name (Text)
     - Budget (Text/Number)
     - Deadline (Date)
     - Tech_Stack (Long Text)
     - Status (Single Select)
     - Summary (Long Text)
     - Confidence (Number)
     - Date_Added (Date)
   
   - **Archive Table** columns:
     - RFP_Name (Text)
     - Rejection_Reason (Long Text)
     - Tech_Stack (Long Text)
     - Confidence (Number)
     - Date_Processed (Date)

5. **Google Gemini API Key** (or alternative AI provider)

### Credentials Configuration

Configure these credentials in n8n:

1. **Google Drive API**
   - OAuth2 authentication
   - Permissions: Drive read access

2. **Gmail API**
   - OAuth2 authentication
   - Permissions: Send emails

3. **Baserow API**
   - API token authentication
   - Access to designated workspace

4. **Google Gemini API**
   - API key authentication

---

## 🚀 Installation Steps

### Step 1: Import Workflow
1. Copy the workflow JSON
2. In n8n, go to **Workflows** → **Import from File/URL**
3. Paste the JSON and import

### Step 2: Configure Credentials
1. Click on each node with a credential requirement
2. Select "Create New Credential"
3. Follow authentication prompts

### Step 3: Set Up Google Drive Folder
1. Create a folder named "RFP_Inbox" in Google Drive
2. Note the folder ID from the URL
3. Update the Google Drive node with this folder ID

### Step 4: Configure Baserow Tables
1. Create two tables as specified above
2. Get table IDs from Baserow
3. Update both Baserow nodes with correct table IDs

### Step 5: Customize AI Prompt
1. Open the AI Agent node
2. Modify the prompt to match your company's specific criteria
3. Adjust the "good fit" logic based on your competencies

### Step 6: Test the Workflow
1. Upload a sample RFP PDF to your Google Drive folder
2. Click "Execute Workflow" manually
3. Verify each node executes successfully
4. Check email notification
5. Confirm data appears in Baserow

---

## 📊 Workflow Behavior

### Success Path
1. PDF detected in Google Drive
2. Text extracted successfully
3. AI analyzes content (15-30 seconds)
4. Good fit identified
5. Email sent to approver
6. Workflow pauses for approval
7. Upon approval, saved to Opportunities table
8. Notification sent to sales team

### Rejection Path
1. PDF detected in Google Drive
2. Text extracted successfully
3. AI analyzes content
4. Poor fit identified (wrong tech stack, budget mismatch, etc.)
5. Automatically archived with reasoning
6. No human intervention needed

---

## 🎯 Use Cases

### When to Use This Workflow

✅ **Ideal For:**
- High volume of incoming RFPs
- Need for quick initial screening
- Standardized evaluation criteria
- Teams that value data-driven decisions
- Organizations tracking opportunity pipeline

❌ **Not Ideal For:**
- Low volume of highly customized proposals
- Situations requiring deep domain expertise
- RFPs with complex non-text attachments
- Cases where every opportunity must be manually reviewed

---

## 🔍 Customization Options

### Easy Modifications

1. **Change AI Provider:**
   - Replace Google Gemini node with OpenAI, Claude, or local LLM
   - Update prompt format accordingly

2. **Add Slack Notifications:**
   - Replace Gmail node with Slack
   - Add interactive buttons for inline approval

3. **Multiple Approval Layers:**
   - Add additional IF nodes
   - Create tiered approval (Manager → Director → VP)

4. **Scoring Thresholds:**
   - Modify IF condition to check confidence score
   - Example: Only route if `confidence > 75`

5. **Email Trigger Instead of Manual:**
   - Replace Manual Trigger with Email Trigger (Gmail/Outlook)
   - Automatically process RFPs sent to specific email

---

## 🐛 Troubleshooting

### Common Issues

**Problem:** PDF extraction fails
- **Solution:** Ensure PDF is text-based, not scanned image
- **Alternative:** Add OCR node (Tesseract) before Extract node

**Problem:** AI returns invalid JSON
- **Solution:** Add error handling in Code node:
```javascript
try {
  const parsed = JSON.parse(aiResponse);
  return { json: parsed };
} catch (e) {
  return { json: { error: "Parse failed", raw: aiResponse }};
}
```

**Problem:** Workflow doesn't pause for approval
- **Solution:** Verify Wait node is configured with correct webhook or trigger

**Problem:** Baserow connection fails
- **Solution:** Check API token permissions and table IDs

---

## 📈 Performance Metrics

### Expected Processing Times
- PDF extraction: 2-5 seconds
- AI analysis: 10-30 seconds
- Total automated flow: 15-40 seconds
- Human approval: Variable (typically 1-48 hours)

### Scalability
- Can process: 50-100 RFPs per day
- Concurrent executions: Limited by n8n plan
- Storage: Minimal (text data only)

---

## 🔒 Security Considerations

1. **Data Privacy:** RFP content is sent to external AI provider
2. **Access Control:** Limit Google Drive folder access
3. **API Keys:** Store securely in n8n credentials
4. **Audit Trail:** All decisions logged in Baserow with timestamps

