# SharePoint School Folder Automation Guide
## Power Automate Flow Implementation

### Overview
This guide provides step-by-step instructions for creating 420 school-specific folders in SharePoint with automated permission management using Power Automate Flow.

---

## Phase 1: Information Gathering & Prerequisites

### Required Information Checklist
- [ ] **SharePoint Site URL** (e.g., https://yourorg.sharepoint.com/sites/SchoolData)
- [ ] **Document Library Name** (e.g., "School Folders" or "Documents")
- [ ] **Complete School List** (420 schools with exact names/codes)
- [ ] **Folder Naming Convention** (e.g., "School_001_[SchoolName]" or "[SchoolCode]_[SchoolName]")
- [ ] **Administrator Email Addresses** (who needs full access)
- [ ] **School Contact Information** (email addresses for each school's access)

### Required Permissions
- [ ] **SharePoint Site Collection Administrator** access
- [ ] **Power Platform Administrator** or **Power Automate Premium** license
- [ ] **Azure Active Directory** permissions to create security groups
- [ ] **SharePoint Online Management Shell** (PowerShell module) access

### Tools & Resources Needed
- [ ] **Excel file** with school data (template provided below)
- [ ] **Power Automate** access (Premium license recommended)
- [ ] **SharePoint Online** access
- [ ] **Azure Active Directory** admin center access

---

## Phase 2: Data Preparation

### Excel Data Template
Create an Excel file named `School_Data.xlsx` with the following columns:

| Column Name | Description | Example |
|-------------|-------------|---------|
| SchoolID | Unique identifier | SCH001 |
| SchoolName | Full school name | Washington Elementary |
| FolderName | Desired folder name | SCH001_Washington_Elementary |
| ContactEmail | Primary school contact | principal@washington.edu |
| Region | Geographic region (optional) | North District |
| Status | Active/Inactive | Active |

### Sample Data Format
```
SchoolID,SchoolName,FolderName,ContactEmail,Region,Status
SCH001,Washington Elementary,SCH001_Washington_Elementary,principal@washington.edu,North District,Active
SCH002,Lincoln High School,SCH002_Lincoln_High_School,admin@lincoln.edu,South District,Active
SCH003,Roosevelt Middle School,SCH003_Roosevelt_Middle_School,office@roosevelt.edu,East District,Active
```

### Data Validation Steps
1. **Remove duplicates** from school list
2. **Validate email addresses** are correct and active
3. **Check folder naming** compliance (no special characters: / \ : * ? " < > |)
4. **Verify school count** matches expected 420 schools
5. **Test with 5-10 schools** before full deployment

---

## Phase 3: SharePoint Site Preparation

### Site Structure Setup
1. **Navigate to SharePoint site** where folders will be created
2. **Create or identify target document library**
3. **Create parent folder** (e.g., "School Folders") if needed
4. **Document the site structure**:
   - Site URL: _______________
   - Library Name: _______________
   - Parent Folder Path: _______________

### Security Groups Creation
**Option A: Manual Creation (Small Scale)**
1. Go to **SharePoint Site Settings > People and Groups**
2. Create groups for each school: `[SchoolName]_Access`
3. Add school contacts to respective groups

**Option B: Automated Creation (Recommended)**
1. Use **PowerShell script** to create Azure AD security groups
2. Name convention: `SharePoint_[SchoolName]_Access`
3. Add school contacts to groups via PowerShell

---

## Phase 4: Power Automate Flow Creation

### Flow 1: Folder Creation Flow

#### Step 1: Create New Flow
1. Go to **Power Automate** (make.powerautomate.com)
2. Select **Create > Automated cloud flow**
3. Name: `Create School Folders`
4. Trigger: **When an item is created or modified (SharePoint)**

#### Step 2: Configure Trigger
- **Site Address**: Your SharePoint site URL
- **List Name**: Select your Excel list/SharePoint list with school data

#### Step 3: Add Folder Creation Action
1. **New step > SharePoint > Create new folder**
2. **Site Address**: Your SharePoint site URL
3. **Folder Path**: `/sites/[YourSite]/[DocumentLibrary]/School Folders`
4. **Name**: Use dynamic content `FolderName` from trigger

#### Step 4: Add Permission Setting Action
1. **New step > SharePoint > Send an HTTP request to SharePoint**
2. **Method**: POST
3. **Uri**: `_api/web/GetFolderByServerRelativeUrl('/sites/[YourSite]/[DocumentLibrary]/School Folders/[FolderName]')/ListItemAllFields/breakroleinheritance(copyRoleAssignments=false,clearSubscopes=true)`
4. **Headers**: 
   ```json
   {
     "Accept": "application/json;odata=verbose",
     "Content-Type": "application/json;odata=verbose"
   }
   ```

#### Step 5: Grant Permissions
1. **New step > SharePoint > Send an HTTP request to SharePoint**
2. **Method**: POST
3. **Uri**: `_api/web/GetFolderByServerRelativeUrl('/sites/[YourSite]/[DocumentLibrary]/School Folders/[FolderName]')/ListItemAllFields/roleassignments/addroleassignment(principalid=[GroupID],roleDefId=[RoleDefinitionID])`

### Flow 2: Permission Management Flow

#### Step 1: Create Permission Lookup Flow
1. **New flow**: `Manage School Folder Permissions`
2. **Trigger**: Manual trigger or scheduled
3. **Action**: Update permissions based on changes

#### Step 2: Configure Variables
- **SchoolGroupID**: Get from Azure AD
- **AdminGroupID**: Site administrators group
- **ContributorRoleID**: Typically ID = 3
- **ReaderRoleID**: Typically ID = 2

---

## Phase 5: Testing & Validation

### Testing Checklist
- [ ] **Test with 3-5 schools** first
- [ ] **Verify folder creation** in correct location
- [ ] **Check permissions** are set correctly
- [ ] **Test access** with school contact accounts
- [ ] **Verify admin access** to all folders
- [ ] **Test permission inheritance** is broken properly

### Validation Steps
1. **Login as school contact** and verify access to only their folder
2. **Login as administrator** and verify access to all folders
3. **Check folder structure** matches naming convention
4. **Verify no duplicate folders** were created
5. **Test file upload/download** permissions

---

## Phase 6: Full Deployment

### Deployment Process
1. **Backup current SharePoint site** (if applicable)
2. **Run flow for all 420 schools** in batches of 50-100
3. **Monitor flow execution** for errors
4. **Document any failures** and retry
5. **Generate completion report**

### Monitoring & Maintenance
- **Set up flow monitoring** for failures
- **Create process** for adding new schools
- **Document permission changes** procedure
- **Schedule regular audits** of folder permissions

---

## Phase 7: Documentation & Handover

### Documentation Required
1. **Flow configuration** screenshots and settings
2. **Permission matrix** (who has access to what)
3. **Troubleshooting guide** for common issues
4. **User access instructions** for school contacts
5. **Administrator maintenance** procedures

### Training Materials
- **End-user guide** for school contacts
- **Administrator guide** for IT staff
- **Troubleshooting steps** for common issues
- **Contact information** for support

---

## Troubleshooting Guide

### Common Issues & Solutions

**Issue**: Flow fails with "Access Denied" error
**Solution**: Verify flow has proper SharePoint permissions and site collection admin access

**Issue**: Folders created but permissions not set
**Solution**: Check Azure AD group IDs and role definition IDs in flow

**Issue**: Duplicate folders created
**Solution**: Add condition to check if folder exists before creation

**Issue**: Email addresses not found in Azure AD
**Solution**: Verify email addresses are correct and users exist in tenant

**Issue**: Permission inheritance not broken
**Solution**: Check SharePoint REST API call syntax and site URL format

### Support Contacts
- **SharePoint Administrator**: _______________
- **Power Platform Administrator**: _______________
- **IT Help Desk**: _______________

---

## Security Considerations

### Best Practices
1. **Use service accounts** for automation where possible
2. **Implement least privilege** access principles
3. **Regular permission audits** (quarterly recommended)
4. **Monitor flow execution** logs for suspicious activity
5. **Document all changes** to folder structure

### Compliance Requirements
- **Data retention policies** may apply
- **Audit logging** should be enabled
- **Regular access reviews** required
- **Incident response** procedures documented

---

## Appendix

### PowerShell Scripts (Alternative Method)
```powershell
# Sample script for bulk folder creation
Connect-PnPOnline -Url "https://yourorg.sharepoint.com/sites/SchoolData"
$schools = Import-Csv "School_Data.csv"
foreach ($school in $schools) {
    Add-PnPFolder -Name $school.FolderName -Folder "School Folders"
}
```

### Useful SharePoint REST API Endpoints
- **Create Folder**: `_api/web/folders/add('/sites/[site]/[library]/[path]')`
- **Break Inheritance**: `_api/web/GetFolderByServerRelativeUrl('[path]')/ListItemAllFields/breakroleinheritance`
- **Add Role Assignment**: `_api/web/GetFolderByServerRelativeUrl('[path]')/ListItemAllFields/roleassignments/addroleassignment`

### Contact Information
**Project Lead**: _______________
**Implementation Date**: _______________
**Next Review Date**: _______________

---

*This guide should be updated as requirements change or new features are added to the automation process.*
