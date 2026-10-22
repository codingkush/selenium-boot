---
description: "Read HTML table rows with Selenium Boot Locator.rows(), find rows by column value, assert cells, and handle short rows and group headings."
id: read-table-rows
title: Read table rows
sidebar_label: Read table rows
---

# Read table rows

Read an HTML table as data instead of walking its cells by hand. Call `rows()` on a
locator that points at the `<table>`, and you get back one map per row, keyed by
column header.

## Read, find, and assert on rows

Given a table like this:

```html
<table id="table1">
  <thead>
    <tr><th>Last Name</th><th>First Name</th><th>Due</th></tr>
  </thead>
  <tbody>
    <tr><td>Smith</td><td>John</td><td>$50.00</td></tr>
    <tr><td>Doe</td><td>Jason</td><td>$100.00</td></tr>
  </tbody>
</table>
```

read it, find a row by a column's value, and assert on a cell:

```java title="AccountsTableTest.java"
import com.seleniumboot.test.BaseTest;
import java.util.List;
import java.util.Map;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class AccountsTableTest extends BaseTest {
 
    @Test
    public void doeOwesOneHundred() {
        open("/accounts");
 
        // One map per row, keyed by column header:
        // [{Last Name=Smith, First Name=John, Due=$50.00},
        //  {Last Name=Doe,   First Name=Jason, Due=$100.00}]
        List<Map<String, String>> rows = $("#table1").rows();
 
        // Find a row by a column's value
        Map<String, String> doe = rows.stream()
                .filter(row -> row.get("Last Name").equals("Doe"))
                .findFirst()
                .orElseThrow();
 
        // Assert on a cell by its column header
        Assert.assertEquals(doe.get("Due"), "$100.00");
        // fails with: expected [$100.00] but found [$90.00] if the cell says $90.00
    }
}
```

`rows()` returns a plain `List`, so finding a row is ordinary Java: a stream as above,
or a `for` loop if you prefer.

For each example below, reuse `#table1` and its `<thead>`, replacing only the `<tbody>`.

## Short rows

When a row has fewer `<td>` cells than there are headers, `rows()` retains every
header key and fills missing trailing values with `""`. Here, Smith's final
cell is missing, while Doe's row is complete:

```html
<tbody>
  <tr><td>Smith</td><td>John</td></tr>
  <tr><td>Doe</td><td>Jason</td><td>$100.00</td></tr>
</tbody>
```

```java
List<Map<String, String>> rows = $("#table1").rows();
Map<String, String> smith = rows.get(0);
Assert.assertTrue(smith.containsKey("Due"));
Assert.assertEquals(smith.get("Due"), "");
// Smith retains the Due key with an empty-string value.
Assert.assertEquals(rows.get(1).get("Due"), "$100.00");
// Doe's complete row keeps its original value.
```

Note that cells are matched by position. Omitting a middle `<td>` shifts later
values to earlier columns; use an empty `<td></td>` to preserve the position.
Cells beyond the header count are ignored.

## Group rows containing only `<th>`

Once the column headers are selected, rows containing no `<td>` cells are
skipped. The group heading below spans three columns but is not a data row:

```html
<tbody>
  <tr><th colspan="3">Outstanding accounts</th></tr>
  <tr><td>Smith</td><td>John</td></tr>
  <tr><td>Doe</td><td>Jason</td><td>$100.00</td></tr>
</tbody>
```

```java
List<Map<String, String>> rows = $("#table1").rows();
Assert.assertEquals(rows.size(), 2);
Assert.assertEquals(rows.get(0).get("Last Name"), "Smith");
Assert.assertEquals(rows.get(1).get("Last Name"), "Doe");
// The group heading is skipped; Smith and Doe remain in order.
```

Note that header selection matters: without a `<thead>` header, the first row
containing `<th>` supplies the keys. Keep the column headers in `<thead>` so a
preceding group heading is not selected as the header.

**Deeper reference:** [Accessibility-First Locators](/docs/guides/semantic-locators) — the `$()` / `Locator` API that `rows()` belongs to.
