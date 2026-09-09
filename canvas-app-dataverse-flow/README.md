# Build a Canvas Power App with a Dataverse Table and a Power Automate Flow

**Duration:** ~35 minutes
**Level:** Beginner
**Product area:** Microsoft Power Platform (Power Apps, Dataverse, Power Automate)

## Scenario

You work for **Contoso**, and the facilities team needs a quick way to log
maintenance requests. In this lab you will:

1. Create a custom **Dataverse** table to store requests.
2. Build a **Canvas Power App** to add and view requests.
3. Create a **Power Automate cloud flow** and call it from the app to send a
   confirmation notification.

---

## Part 1 — Create the Dataverse table (~8 min)

1. Go to **https://make.powerapps.com** and confirm your environment
   (top-right environment picker).
2. In the left nav, select **Tables** → **+ New table** →
   **Table (advanced properties)** (this opens the properties panel where you can
   set the display name).

   ![Creating a new Dataverse table from the Tables area](images/P1-new-table.png)

3. In the properties panel enter:
   - **Display name:** `Maintenance Request`
   - Plural name auto-fills to `Maintenance Requests`.
4. Select **Save**. Dataverse creates the table with a primary column
   **Name** (rename its display name to `Title` via the column settings if you like).
5. Select the **+** next to the columns list to open the **New column** panel.

   ![Opening the New column panel with the + button](images/P1-new-column.png)

6. Add these columns by defining **Display name**, **Data type**, and **Format**.

   | Display name  | Data type            | Notes                                            |
   |---------------|----------------------|--------------------------------------------------|
   | `Description` | Multiline text       | Details of the issue                             |
   | `Location`    | Single line of text  | Building / room                                  |
   | `Priority`    | Choice               | Choices: `Low`, `Medium`, `High`                 |
   | `Status`      | Choice               | Choices: `New`, `In Progress`, `Done` (default `New`) |
   | `Requestor Email` | Single line of text (Email format) | Who logged it                       |

   > [!CAUTION]
   > **Choice columns:** For `Priority` and `Status`, when the **New column**
   > panel asks **Sync with global choice?**, set it to **No** so you create a
   > local choice with your own options for this table.

   ![Sync with global choice set to No](images/sync-global-choice.png)

7. Your table should now look like this. *(Optional)* Add a couple of sample rows
   via **Edit** → **+ New row**.

   ![Maintenance Request table with custom columns](images/P1-table-columns.png)

✅ **Checkpoint:** You have a `Maintenance Request` table with custom columns.

### Alternative: use a SharePoint list instead

Prefer a **SharePoint list** (or don't have Dataverse)? You can build the whole lab
on a SharePoint list with the same fields — the app and flow steps are almost
identical (in Power Apps you'd add the list as the data source instead of the
Dataverse table).

1. Open **SharePoint** (Microsoft 365 app launcher → **SharePoint**) and go to the
   site where you want the list.

   ![Opening SharePoint from the app launcher](images/P1-sharepoint.png)
2. Select **Build** → **List**.

   ![Creating a new blank list](images/P1-create-list.png)
3. Select **List** and provide name `Maintenance Requests`, verify that
   **My lists** (OneDrive) is selected and then click **Create list**.

   ![Setting up the new list](images/P1-setup-list.png)
4. When the list is created, click **Go to list**.
5. The list already has a **Title** column (single line of text). Add the rest via
   **+ Add column**:

   | Column name   | Column type            | Notes                                            |
   |---------------|------------------------|--------------------------------------------------|
   | `Description` | Multiple lines of text | Details of the issue                             |
   | `Location`    | Single line of text    | Building / room                                  |
   | `Priority`    | Choice                 | Choices: `Low`, `Medium`, `High`                 |
   | `Status`      | Choice                 | Choices: `New`, `In Progress`, `Done` (default `New`) |
   | `Requestor Email` | Single line of text | Who logged it                                    |

6. List should look like this now. Copy your OneDrive site URL (for example to
   Notepad) as we need the URL in Part 3 when creating the Power App.

   ![The list with all columns ready](images/P1-list-ready.png)

7. *(Optional)* Add a couple of sample items with **+ New**.

> **Notes when using SharePoint instead of Dataverse:**
> - In Power Apps, add data via **Data → + Add data → SharePoint**, pick your site,
>   then the `Maintenance Requests` list.
> - Choice columns return a **record** in Power Fx — reference `.Value`
>   (e.g. `ThisItem.Priority.Value`), same as Dataverse choices.
> - When submitting from an Edit form, the choice value is written back
>   automatically by the form's data card.

---

## Part 2 — Create the Power Automate cloud flow (~10 min)

You'll build the flow first so the app can call it.

1. Go to **https://make.powerautomate.com** (same environment).
2. Select **Create** → **Instant cloud flow**.

   ![Creating a new instant cloud flow](images/P2-new-instant-flow.png)

3. Name it `Notify Maintenance Request`, choose **When Power Apps calls a flow
   (V2)** and then click **Create**.

   ![Naming the flow and choosing the PowerApps V2 trigger](images/P2-new-instant-flow-2.png)

4. On the **PowerApps (V2)** trigger, select **+ Add an input**:
   - Add a **Text** input named `RequestTitle`.
   - Add a **Text** input named `RequestorEmail`.
   - Add a **Text** input named `Priority`.

   ![PowerApps V2 trigger with three text inputs](images/P2-flow-trigger-inputs.png)

5. Select **+ New action**.

   ![Adding a new action to the flow](images/P2-new-action.png)

6. Search for send an email and select **Office 365 Outlook** > **Send an email
   (V2)** action.

   ![Searching for the Send an email action](images/P2-send-an-email-action.png)

7. Select **Sign in** to create the connection.

   ![Signing in to create the Office 365 Outlook connection](images/P2-outlook-connection.png)

8. Configure **Send an email (V2)** action like below:
   - Rename the action as `SendEmail`.

     ![Renaming the action to SendEmail](images/P2-sendanemail-rename.png)
   - Enable Dynamic content for the **To** field.

     ![Enabling Dynamic content for the To field](images/P2-sendanemail-dynamic-content.png)
   - **To:** click lightning icon and select `RequestorEmail`.

     ![Selecting RequestorEmail for the To field](images/P2-sendanemail-to.png)
   - **Subject:** `Request received: ` then insert dynamic content `RequestTitle`.

     ![Setting the Subject with dynamic content](images/P2-sendanemail-subject.png)
   - **Body:** Copy following text to it and use the **Dynamic content** picker
     to insert **RequestTitle** and **Priority** instead of typing the tokens.
     ```
     Your maintenance request "@{triggerBody()['text']}" was logged.
     Priority: (insert Priority dynamic content)
     We'll follow up shortly. — Contoso Facilities
     ```

     ![Inserting Priority dynamic content in the Body](images/P2-sendanemail-priority.png)
9. Add **+ New step** → **Respond to a PowerApp or flow**.

   ![Respond to a PowerApp or flow action](images/P2-respond-to-app.png)
10. Add output (Text) with name `Result`, then click the value field and select
    expression.

    ![Returning the Result output from the flow](images/P2-respond-to-app-2.png)
11. Type or copy-paste this expression to the field and click **Add**:

    ```
    actions('SendEmail').status
    ```

    ![Adding the SendEmail status expression](images/P2-respond-to-app-3.png)
12. **Save** the flow.

   ![Saving the flow](images/P2-save-flow.png)

✅ **Checkpoint:** A flow named `Notify Maintenance Request` accepts 3 inputs and
sends an email.

---

## Part 3 — Build the Canvas app (responsive) (~12 min)

1. Back in **https://make.powerapps.com/**, select **+ Create** → **Create from
   blank**.

   ![Create from blank in Power Apps](images/P4-create-app.png)
2. Select **Responsive**.

   ![Selecting the Responsive app option](images/P4-responsive-app.png)

3. Click **+** in header area and select **Text label**.

   ![Adding a Text label to the header](images/P4-add-header.png)
4. Configure control in properties pane like below:
   - Text: `Maintenance Request`
   - Font: 24
   - Text alignment: Center
   - Auto height: On
   - Flexible width: On

   ![Configuring the header label](images/P4-header-conf.png)
5. Select **MainContainer1** from the tree view and change its
   **Direction → Horizontal**.

   ![Setting MainContainer1 direction to Horizontal](images/P3-main-container.png)
6. **Connect the data:** left rail **Data** → **+ Add data** → search/select
   `Maintenance Request`. **NOTE!** If you are using a List then jump to the next
   step.

   ![Adding the Maintenance Request Dataverse data source](images/P4-add-data.png)

7. **NOTE!** Skip this if using Dataverse. When using a **List** (SharePoint),
   search the **SharePoint** connector and select it. Then select an existing
   connection or create a new one. Provide your OneDrive site URL (the one you
   copied in Part 1) and pick the `Maintenance Requests` list.

   ![Selecting the SharePoint connector](images/P3-spo-data.png)

   ![Choosing or creating the SharePoint connection](images/P3-spo-conn.png)

   ![Providing the OneDrive site URL](images/P3-spo-conn-2.png)

   ![Picking the Maintenance Requests list](images/P3-spo-conn-3.png)

8. Click **+** in the main area (main container) and add **Vertical gallery** control.

   ![Adding a vertical gallery control](images/P4-add-gallery.png)
9. Click **Data** and select **Maintenance Requests**.

   ![Selecting the gallery data source](images/P4-select-gallery-data.png)
10. Click **Layout** and select **Title, subtitle and body**.

    ![Selecting the gallery layout](images/P4-gallery-layout.png)
11. Select Fields and map **Subtitle** and **Body** fields:
    - Subtitle → `ThisItem.Location`
    - Body → `ThisItem.Priority`

    ![Mapping the gallery Subtitle and Body fields](images/P3-gallery-subtitle-body.png)

    > **Using a List (SharePoint)?** Map the same fields — for the choice field
    > use `ThisItem.Priority.Value`.

    ![Mapping the gallery fields for a SharePoint list](images/P3-spo-gallery-fields.png)
12. For responsiveness, avoid using fixed width and height. Set these properties:
    - Align in container: **Stretch**
    - Width: `Parent.Width * 0.5`

    ![Setting the gallery align and width](images/P3-gallery-size.png)

13. Select main container from tree view and add **Edit form**.

    ![Adding an Edit form to the main container](images/P3-add-form.png)
14. Select **Data** and choose **Maintenance Requests**.

    ![Selecting the form data source](images/P3-form-data.png)
15. Remove all the other fields **except**: `Title`, `Description`, `Priority`,
    `Location`.

    ![Keeping only the required form fields](images/P3-form-fields.png)
16. Set form properties:
    - Columns: **1**
    - Default mode: **New**
    - Item: **Gallery1.Selected**

    ![Setting the form properties](images/P3-form-props.png)
17. Click **+** in the footer container (FooterContainer1) and add **Button**
    control.

    ![Adding a Button to the footer container](images/P3-add-submit-button.png)
18. Configure button like below:
    - Text: **Submit**
    - OnSelect:
      ```powerfx
      SubmitForm(Form1)
      ```

    ![Configuring the Submit button](images/P3-submit-button-conf.png)

> 💡 **Responsive tips:** Use **layout containers** instead of absolute
> positioning; size controls relative to `Parent.Width`/`Parent.Height`; and
> preview at different sizes with the browser dev tools device toolbar (F12) to
> confirm the layout reflows.

---

## Part 4 — Call the flow from the app (~5 min)

1. Click three dots from the main left menu and select **Power Automate**.

   ![Opening the Power Automate pane](images/P4-add-flow.png)
2. Click **+ Add flow** and select **Notify Maintenance Request**.

   ![Adding the Notify Maintenance Request flow](images/P4-add-flow-2.png)
3. Select the Form control (Form1) and set its **OnSuccess** event to call the
   flow with the form values:
   ```powerfx
   NotifyMaintenanceRequest.Run(
       Form1.LastSubmit.Title,
       User().Email,
       Form1.LastSubmit.Priority.Value
   );
   Notify("Request submitted and notification sent!", NotificationType.Success);
   ResetForm(Form1)
   ```

   ![Setting the form OnSuccess to call the flow](images/P4-form-onsuccess.png)
   > `OnSuccess` fires only after the record saves, and `Form1.LastSubmit` holds
   > the saved record (unlike `Form1.Updates`, which is cleared by `SubmitForm`).
   > `User().Email` is used because the **Requestor Email** field isn't on the form.
4. **Save** (Ctrl+S) and then **Preview** the app (F5 / ▶ Play).

✅ **Checkpoint:** Submitting the form creates a Dataverse row **and** triggers the
flow, which sends the confirmation email.

---

## Test end-to-end

1. In Preview, fill the form: Title = `Broken AC`, Location = `Bldg 3 / Rm 210`,
   Priority = `High`.
2. Select **Submit** → the success banner appears.
3. Confirm the new record shows in the gallery.
4. Check your inbox for the `Request received: Broken AC` email.
5. In Power Automate → **My flows** → `Notify Maintenance Request` →
   **Run history** to confirm a successful run.

---

## Troubleshooting

- **Flow not listed in the app:** Ensure the flow's trigger is
  *When Power Apps calls a flow (V2)* and it's **saved**; refresh the app.
- **`.Value` errors on Priority/Status:** Choice columns return a record —
  use `.Value` (e.g., `ThisItem.Priority.Value`,
  `Form1.Updates.Priority.Value`).
- **No email received:** Verify the Office 365 Outlook connection is authorized;
  check the flow **Run history** for errors; look in Junk.
- **Form field names with spaces:** Reference them with single quotes,
  e.g., `Form1.Updates.'Requestor Email'`.
- **Permission errors:** Confirm you're in the correct environment and have
  maker/creator rights.

---

## What you learned

- Creating a **custom Dataverse table** with typed and choice columns.
- Building a **responsive Canvas app** (Responsive layout + layout containers)
  that reads (gallery) and writes (edit form) Dataverse data.
- Authoring an **instant Power Automate cloud flow** with PowerApps (V2) inputs.
- **Calling the flow from Power Fx** using `<FlowName>.Run(...)` and passing form values.

## Stretch goals (optional)

- Return a value from the flow and display it with a label.
- Add a **Status** update screen using a second form (`FormMode.Edit`).
- Replace the email action with a **Teams** "Post message" action.
- Add search/filter to the gallery:
  `Filter(Search('Maintenance Requests', TextInput1.Text, "cr_title"), Status.Value = "New")`.
