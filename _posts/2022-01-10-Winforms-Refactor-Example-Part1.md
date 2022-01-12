
### Reasons to refactor

I'm including this **real** code because its a good example of why its so important to refactor. Its part of windows form partial class which is 4000 lines of code long.

I chopped pieces out so your device doesn't lag.

This function Calculates totals, updates the UI form, and the Grid, and it is called from 18 different places in the app.

I included it all now as I only show smaller pieces while I'm explaining a step by step process. This process is mine, if you want something more detailed then take a look at Martin Fowlers book [Refactoring, improving the design of existing code](https://www.informit.com/store/refactoring-improving-the-design-of-existing-code-9780134757599 "Refactoring") or look around for a cheaper copy or earlier edition off amazon.


```
private void CalculateTotals()
{

    if (bsTickets.Count > 0)
    {
        Tax GST = new Tax{ Rate = 0.05M };
        Tax PST = new Tax{ Rate = 0.05M };
        Tax HST = new Tax{ Rate = 0.12M };

        decimal tmp;
        string provinceOfWork = cmbProvinceOfWork.SelectedText;
        GST = currentTicket.GetTaxInfo("GST", provinceOfWork);
        if (cbGSTExempt.Checked)
            GST.Rate = 0;
        else
        {
            tmp = currentTicket.GetTicketTaxRate("GST", currentTicket.TicketID);
            if (tmp != -1)
                GST.Rate = tmp;
        }
        //... code removed for brevity

        DataTable dtTickets = (DataTable)bsTickets.DataSource;
        DataRow[] bindingRow = dtTickets.DataSet.Tables["Tickets"].Select("TicketID = '" + currentTicket.TicketID + "'");

        //clear our tax array as we are going to recalc
        for (int i = 0; i < currentTicket.totalTax.Count(); i++)
            currentTicket.totalTax[i] = 0;
        int billingID;
        billingID = (int)bindingRow[0]["BillingRateID"];
        int safety; //0 = none 1 = field only 2 = full
        safety = (int)bindingRow[0]["safety"];
        dgvTotals.Rows.Clear();

        //add all our labour details
        decimal labourTotalReg = 0, labourTotalOT = 0, vehicleTotalKM = 0, vehicleTotalHours = 0, materialOtherTotal = 0, safetyTotal = 0, susTotal = 0, csTotal = 0;

        for (int i = 0; i < global_dsLabourCharges.Tables["LabourVehicleCharges"].Rows.Count; i++)
        {
            DataRow row = global_dsLabourCharges.Tables["LabourVehicleCharges"].Rows[i];

            if (row.RowState != DataRowState.Deleted)
            {
                decimal regTime, regTimeCharged, overTime, OverTimeCharged, kmsCharged = 0, vehicleHoursCharged = 0, safetyCharged = 0, subCharged = 0, compCharged = 0;

                if (row["RegularHours"] == DBNull.Value)
                    regTime = 0;
                else
                    regTime = (decimal)row["RegularHours"];
                //... code removed for brevity

                //lookup the customer to get the safety rate
                BillingRate rate = new BillingRate
                {
                    //setting the rate also gets the rate info
                    //rate.BillingRateID = billingID;
                    Labour = (double)currentTicket.GetTicketRate("Labour", currentTicket.TicketID),
                    OverTime = (double)currentTicket.GetTicketRate("OverTime", currentTicket.TicketID),
                    VehicleKM = (double)currentTicket.GetTicketRate("VehicleKM", currentTicket.TicketID),
                    VehicleHourly = (double)currentTicket.GetTicketRate("VehicleHourly", currentTicket.TicketID),
                    Sub = (double)currentTicket.GetTicketRate("Sub", currentTicket.TicketID),
                    CS = (double)currentTicket.GetTicketRate("CS", currentTicket.TicketID),
                    Safety = (double)currentTicket.GetTicketRate("Safety", currentTicket.TicketID)
                };

                labourTotalReg = labourTotalReg + (Math.Round(regTimeCharged * (decimal)rate.Labour, 2, MidpointRounding.AwayFromZero));
                //... code removed for brevity
            }

        }

        //add all our material details
        for (int i = 0; i < global_dsMaterialCharges.Tables["MaterialOtherCharges"].Rows.Count; i++)
        {
            DataRow row = global_dsMaterialCharges.Tables["MaterialOtherCharges"].Rows[i];
            if (row.RowState != DataRowState.Deleted)
                materialOtherTotal = materialOtherTotal + ((int)row["Qty"] * (decimal)row["UnitPrice"]);
        }

        //now we have everything totaled. Figure out the GST/PST and sub total info
        DataGridViewRow tRow = new DataGridViewRow();

        DataGridViewCell cell = new DataGridViewTextBoxCell
        {
            Value = "Labour"
        };
        tRow.Cells.Add(cell);

        cell = new DataGridViewTextBoxCell
        {
            Value = labourTotalReg + labourTotalOT
        };
        currentTicket.TotalLabour = labourTotalReg + labourTotalOT;
        tRow.Cells.Add(cell);

        dgvTotals.Rows.Add(tRow);

        //... code removed for brevity

        decimal subTotal = labourTotalReg + labourTotalOT + vehicleTotalKM + vehicleTotalHours + materialOtherTotal + safetyTotal + susTotal + csTotal;
        decimal gstSubTotal = 0;
        gstSubTotal += materialOtherTotal;
        if (GST.ChargeReg)
            gstSubTotal += labourTotalReg;
        //... code removed for brevity

        tRow = new DataGridViewRow();
        cell = new DataGridViewTextBoxCell();

        DataGridViewCellStyle boldCellStyle = new DataGridViewCellStyle
        {
            Font = new Font(dgvTotals.Font, FontStyle.Bold)
        };
        cell.Style = boldCellStyle;
        cell.Value = "    SUBTOTAL";
        tRow.Cells.Add(cell);

        cell = new DataGridViewTextBoxCell
        {
            Style = boldCellStyle,
            Value = subTotal
        };
        tRow.Cells.Add(cell);

        dgvTotals.Rows.Add(tRow);
        //taxes

        decimal gst = 0;
        if ((int)bindingRow[0]["GSTStat"] == 1)
        {

            gst = Math.Round(gstSubTotal * GST.Rate, 2, MidpointRounding.AwayFromZero);
            tRow = new DataGridViewRow();
            cell = new DataGridViewTextBoxCell();
            Log.Write("GST = " + gst);
            cell.Value = "GST";
            tRow.Cells.Add(cell);

            cell = new DataGridViewTextBoxCell();
            currentTicket.totalTax[0] = gst;
            cell.Value = gst;
            tRow.Cells.Add(cell);

            dgvTotals.Rows.Add(tRow);
        }

        decimal pstSubTotal = 0;
        pstSubTotal += materialOtherTotal;
        if (PST.ChargeReg)
            pstSubTotal += labourTotalReg;
        if (PST.ChargeOT)
            pstSubTotal += labourTotalOT;
        if (PST.ChargeKM)
            pstSubTotal += vehicleTotalKM;
        if (PST.ChargeHours)
            pstSubTotal += vehicleTotalHours;
        if (PST.ChargeSafety)
            pstSubTotal += safetyTotal;
        if (PST.ChargeSub)
            pstSubTotal += susTotal;
        if (PST.ChargeComp)
            pstSubTotal += csTotal;

        decimal pst = 0;

        //... code removed for brevity

        tRow = new DataGridViewRow();
        cell = new DataGridViewTextBoxCell
        {
            Style = boldCellStyle,
            Value = "    TOTAL"
        };
        tRow.Cells.Add(cell);

        cell = new DataGridViewTextBoxCell
        {
            Style = boldCellStyle,
            Value = subTotal + gst + pst + hst
        };
        tRow.Cells.Add(cell);

        dgvTotals.Rows.Add(tRow);


        Log.Write("Update totals in Grid");
        if (bindingRow[0][17] == DBNull.Value)
            bindingRow[0][17] = 0;
        if ((decimal)bindingRow[0][17] != subTotal + gst + pst + hst) //total
            bindingRow[0][17] = subTotal + gst + pst + hst; //total
        if (bindingRow[0][14] != DBNull.Value)
            if ((decimal)bindingRow[0][14] != subTotal) //sub total
                bindingRow[0][14] = subTotal; //sub total
        if (bindingRow[0][15] != DBNull.Value)
            if ((decimal)bindingRow[0][15] != gst) //gst
                bindingRow[0][15] = gst; //gst
        if (bindingRow[0][16] != DBNull.Value)
            if ((decimal)bindingRow[0][16] != pst) //pst
                bindingRow[0][16] = pst; //pst

        currentTicket.UpdateTotals();

    }
}

```

> You must use a versioning system, if you don't want to loose all your work - I use git for this blog and my programming, git runs on the desktop and a server version i.e. GitHub if you want it to survive a pc crash. (go find an online tutorial.)

So the *first thing* to do is simply remove the commented out code. It doesn't change behavior. Git will remember the code.

*Next Step* simply shorten each line that is too long or use a format you are more comfortable reading. C# allows code lines to wrap in most cases. Re-build the code to test for errors.

>If you have tests already for your code run them first and then after each change. If you don't have any then consider writing as you go.

Create a new variable wherever an existing function variable is repurposed. Its almost impossible to move these if you don't. You can see them by the = operator (reassigned) being used often, and they often have a name like tmp. I missed or found it hard to follow some before so I address this shortly.

```
int tmp;
tmp = var1 + var2;
var3 = tmp;
tmp = var4 + var5;
etc...
```
> Global variables need to be turned into local variables (i.e. passed in as a parameter), but be careful if an assignment is made - to keep the existing behavior be sure to reassign the Global. I deal with Global variables later, when I plan on moving the Calculations out of the class into another that makes more sense.  

#### 1:
I choose to separate the code as best I can *without* making logical changes.
I called the first part **Bubble-Up** as it reminds me of a bubble sort.

> If a variable is changed anything below the assignment that uses that variable must remain below and everything above the assignment that uses it must remain above.

1. So first I needed to identify where the first time UI is written to. Writing to the UI will go last - makes sense as this function calculates data into totals and write them to the UI.
2. BUT, reading from the UI is likely to be part of the data we need first.
3. Calculating totals will go in the middle.

I found a place where the logic seems to divide.

In this example just after the comment
```
//now we have everything totaled. Figure out the GST/PST and sub total info
```
is the best place to start. To keep it obvious I replaced it with
```
//BUBBLE UP
//end BUBBLE UP
```
Then from the code below we search out the code that is calculated from *untouched* variables.

A few lines down in the code we find
> currentTicket.TotalLabour = labourTotalReg + labourTotalOT;

Both labourTotalReg and labourTotalOT are used but not assigned to just before that. currentTicket.TotalLabour is not used before that and is a total that is being assigned to not added to just yet.

We can move it up between our new comments.
Lets do the same for the rest.

Ok I spent some time and identified the code below could all be safely move together without changing behavior
```
//BUBBLE UP
currentTicket.TotalLabour = labourTotalReg + labourTotalOT;
currentTicket.TotalVehicle = vehicleTotalKM + vehicleTotalHours;
currentTicket.Totalsafety = safetyTotal;
currentTicket.TotalMaterial = materialOtherTotal;
decimal subTotal = labourTotalReg + labourTotalOT + vehicleTotalKM + vehicleTotalHours +   materialOtherTotal + safetyTotal + susTotal + csTotal;
decimal gstSubTotal = 0;
gstSubTotal += materialOtherTotal;
if (GST.ChargeReg)
    gstSubTotal += labourTotalReg;
if (GST.ChargeOT)
    gstSubTotal += labourTotalOT;
if (GST.ChargeKM)
    gstSubTotal += vehicleTotalKM;
if (GST.ChargeHours)
    gstSubTotal += vehicleTotalHours;
if (GST.ChargeSafety)
    gstSubTotal += safetyTotal;
if (GST.ChargeSub)
    gstSubTotal += susTotal;
if (GST.ChargeComp)
    gstSubTotal += csTotal;

decimal gst = 0;
int gstStat = (int)bindingRow[0]["GSTStat"];

if (gstStat == 1)
{
    gst = Math.Round(gstSubTotal * GST.Rate, 2, MidpointRounding.AwayFromZero);
    currentTicket.totalTax[0] = gst;
}
decimal pstSubTotal = 0;
pstSubTotal += materialOtherTotal;
if (PST.ChargeReg)
    pstSubTotal += labourTotalReg;
if (PST.ChargeOT)
    pstSubTotal += labourTotalOT;
if (PST.ChargeKM)
    pstSubTotal += vehicleTotalKM;
if (PST.ChargeHours)
    pstSubTotal += vehicleTotalHours;
if (PST.ChargeSafety)
    pstSubTotal += safetyTotal;
if (PST.ChargeSub)
    pstSubTotal += susTotal;
if (PST.ChargeComp)
    pstSubTotal += csTotal;

decimal pst = 0;
int pstStat = Convert.ToInt32(bindingRow[0]["PSTStat"]);

if (pstStat == 1)
{
    pst = Math.Round(pstSubTotal * PST.Rate, 2, MidpointRounding.AwayFromZero);
    currentTicket.totalTax[1] = pst;
}

decimal hstSubTotal = 0;
hstSubTotal += materialOtherTotal;
if (HST.ChargeReg)
    hstSubTotal += labourTotalReg;
if (HST.ChargeOT)
    hstSubTotal += labourTotalOT;
if (HST.ChargeKM)
    hstSubTotal += vehicleTotalKM;
if (HST.ChargeHours)
    hstSubTotal += vehicleTotalHours;
if (HST.ChargeSafety)
    hstSubTotal += safetyTotal;
if (HST.ChargeSub)
    hstSubTotal += susTotal;
if (HST.ChargeComp)
    hstSubTotal += csTotal;

decimal hst = 0;
int hstStat = Convert.ToInt32(bindingRow[0]["HSTStat"]);
if (hstStat == 1)
{
    hst = Math.Round(hstSubTotal * HST.Rate, 2, MidpointRounding.AwayFromZero);
    currentTicket.totalTax[2] = hst;
}

decimal totalWithTax = subTotal + gst + pst + hst;
//END BUBBLE UP
```


Woah! would you explain that?
Well I just did, each bit has been extracted and moved up in the same order as I found it. It took me an hour or so to do. Plus I did one change at a time, recompiled and ran the app looking at the fields updated to see any visible display differences.

There is some code after this which uses some of the calculate variables which can now be replaced by the calculated values. i.e.
```
Value = labourTotalReg + labourTotalOT
//with
Value = currentTicket.TotalLabour
```

I got rid of the unnecessary lines of log writing (they are strewn all over and write physical files to the hard-drive. A common practice for initial development or ridiculously illusive bugs, but generally there is no place for them in production code).

>Test, and in git commit your changes to your feature branch. Large changes between saves/steps will lead to bugs. And you will have to repeat all the other code in that commit too. Over the years, I've wasted months by trying to go too fast. Don't do it.

> If you rename variables use the refactor/rename in the Editor (my case Visual Studio community) and make sure you have a good reason.

#### 2:

So now there is still more to do.

The if statement wrapping the code and returning nothing bugged me so I inverted it and made it do an early return to get out of the function and to remove one level of depth.

```
if (bsTickets.Count <= 0)
{
   return;
}
```

Then I found some code that was repeated more than once (in more than two places).

The place I'm referring to adds rows to a table that will be used to display the calculated totals. I created a function copying the code that was repeated in each piece and give it a reasonable name.

```
private void CreateAndAddActualRows(DataGridView actualTable, string name, decimal value, FontStyle style = FontStyle.Regular)
{
    DataGridViewRow tRow = new DataGridViewRow();
    DataGridViewCellStyle cellStyle;
    DataGridViewCell cell1;
    DataGridViewCell cell2;
    cellStyle = new DataGridViewCellStyle
    {
        Font = new Font(dgvActuals.Font, style)
    };
    cell1 = new DataGridViewTextBoxCell
    {
        Style = cellStyle,
        Value = name
    };
    cell2 = new DataGridViewTextBoxCell
    {
        Style = cellStyle,
        Value = value
    };

    tRow.Cells.Add(cell1);
    tRow.Cells.Add(cell2);
    actualTable.Rows.Add(tRow);

}
```
Then I made changes to call the new code. Each one was tested by simply running the code/app and viewing the results. (unit testing would be faster and less bug prone)

```
//UI BUILD ROWS OF TOTALS/ACTUALS

CreateAndAddActualRows(dgvActuals, "Labour", currentTicket.TotalLabour);
CreateAndAddActualRows(dgvActuals, "Vehicle", currentTicket.TotalVehicle);
CreateAndAddActualRows(dgvActuals, "Subsistence", susTotal);
CreateAndAddActualRows(dgvActuals, "Computer/Software", csTotal);
CreateAndAddActualRows(dgvActuals, "Remote Support", safetyTotal);
CreateAndAddActualRows(dgvActuals, "Material/Other", materialOtherTotal);
CreateAndAddActualRows(dgvActuals, "    SUBTOTAL", subTotal, FontStyle.Bold);
if (gstStat == 1)
    CreateAndAddActualRows(dgvActuals, "GST", gst);
if (pstStat == 1)
    CreateAndAddActualRows(dgvActuals, "PST", pst);
if (hstStat == 1)
    CreateAndAddActualRows(dgvActuals, "HST", hst);
CreateAndAddActualRows(dgvActuals, "    TOTAL", totalWithTax, FontStyle.Bold);

```

The grid rows were being updated manually and are ugly so I simply extracted them. Note objects are passed by reference.
If you bubbled-up this code too just below the calculations, then call the function from there. they are simple assignments of totals as is much of the rest of the code so can be anywhere below the calculation of the totals.
```

private void UpdateBindingRows(DataRow bindingRow, Decimal total, decimal subtotal, decimal gst, decimal pst, decimal hst)
{
    //UPDATE TOTALS in binding row
    Log.Write("Update totals in Grid");
    if (bindingRow[17] == DBNull.Value)
        bindingRow[17] = 0;
    if ((decimal)bindingRow[17] != total) //total
        bindingRow[17] = total; //total
    if (bindingRow[14] != DBNull.Value)
        if ((decimal)bindingRow[14] != subtotal) //sub total
            bindingRow[14] = subtotal; //sub total
    if (bindingRow[15] != DBNull.Value)
        if ((decimal)bindingRow[15] != gst) //gst
            bindingRow[15] = gst; //gst
    if (bindingRow[16] != DBNull.Value)
        if ((decimal)bindingRow[16] != pst) //pst
            bindingRow[16] = pst; //pst
}
```

Not elegant, but it needs its own function. The original function at the top is starting to look better.

#### 3:

The Taxes GST,PST,& HST use a reused tmp. (found in the else statements near the top). I mentioned this before, so time to get rid of it.

The top definition is removed and they are redefined in scope.

`//decimal tmp;`  then fix the places that now fail when building.
i.e.
```
decimal tmp = currentTicket.GetTicketTaxRate("GST", currentTicket.TicketID);
```

and code like
```
//clear our tax array as we are going to recalc
for (int i = 0; i < currentTicket.totalTax.Count(); i++)
    currentTicket.totalTax[i] = 0;
```
can be changed to by making the Clear Tax Totals function where the variables are.
```
//we are going to recalc
currentTicket.ClearTaxTotals();
```

the code is looking much better.

### Next Post:
We remove code into its own functions and move them to other classes where they belong. We find globals (they are even have global in their names to make them obvious) that can cause problems and change them to be parameters, or if possible local values.
We also deal with unused variables and variables that are incorrectly used from elsewhere. Plus finally we get to the point where we need to turn the calculations into their own function which can be reused.
