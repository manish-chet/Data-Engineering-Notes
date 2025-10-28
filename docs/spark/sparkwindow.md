
ROW_NUMBER():
    ◦ Assigns a unique, sequential integer to each row within its partition, starting from 1.
    ◦ It does not consider ties in the ordering column; if two rows have the same value, they will still receive different, consecutive row numbers.
    ◦ Analogy: Like giving a unique ID to each student in a class based on their height, even if some have the same height.
• RANK():
    ◦ Assigns a rank to each row within its partition.
    ◦ If there are ties (multiple rows have the same value in the ordering column), they receive the same rank.
    ◦ However, RANK() leaves gaps in the ranking sequence for subsequent rows after a tie. For example, if two employees are rank 1, the next unique salary will get rank 3 (skipping rank 2).
    ◦ Analogy: If two students are the tallest, they both get rank 1, and the next tallest student gets rank 3, as if rank 2 was "taken" by the tied students.
• DENSE_RANK():
    ◦ Similar to RANK(), it assigns the same rank to rows with tied values in the ordering column.
    ◦ The crucial difference is that DENSE_RANK() does not leave gaps in the ranking sequence. If two employees are rank 1, the next unique salary will get rank 2.
    ◦ Analogy: If two students are the tallest, they both get rank 1, and the next tallest student gets rank 2, effectively "compressing" the ranks


| Salary | Row Number | Rank | Dense Rank |
|--------|------------|------|------------|
| 5000   | 1          | 1    | 1          |
| 6000   | 2          | 2    | 2          |
| 7000   | 3          | 3    | 3          |
| 8000   | 4          | 4    | 4          |
| 8000   | 5          | 4    | 4          |
| 8000   | 6          | 4    | 4          |
| 9000   | 7          | 7    | 5          |




LEAD and LAG are window functions that allow you to access data from a preceding (LAG) or succeeding (LEAD) row within the same result set, often within a defined "window" or partition.
• LAG: Used to access data from a previous record. If you are currently in the January record, LAG would let you access the December record (or the record immediately preceding January based on your orderBy clause).
• LEAD: Used to access data from a subsequent record. If you are in the January record, LEAD would let you access the February record (or the record immediately following January).
Both lag and lead functions take three arguments:
1. Column Name: The name of the column from which you want to retrieve the value (e.g., "Sales").
2. Offset: An integer specifying how many records back (for LAG) or forward (for LEAD) you want to look. This is not about months but about the number of rows/records. For example, 1 means one record back/forward, 2 means two records back/forward.
3. Default Value: A value to be returned if there is no record available at the specified offset (e.g., if you ask for a previous record for the very first record in a partition). By default, this value is null. You can set it to 0, 100, or anything else.
Interaction with Window Functions: LEAD and LAG must be used with a window specification. A window function defines a "frame" or "partition" of rows on which the function will operate. This is crucial because LEAD and LAG only operate within their designated window and cannot cross partition boundaries.
A typical window specification for these functions involves:
• partitionBy: This divides your data into logical groups (e.g., by product_id or product_name). LEAD and LAG operations will reset for each new partition. For example, if you partition by product, an iPhone's previous month sales will only look at previous iPhone sales, not Samsung sales.
• orderBy: This specifies the order of rows within each partition (e.g., by sales_data or month). This ordering is critical for lag and lead to correctly identify "previous" or "next" records



Window Frame Components:
        ▪ UNBOUNDED PRECEDING: Refers to all rows from the start of the window.
        ▪ UNBOUNDED FOLLOWING: Refers to all rows from the current row to the end of the window.
        ▪ CURRENT ROW: Refers to the row currently being processed.
        ▪ ROWS BETWEEN: This function defines the start and end boundaries of the window frame based on physical row offsets.
            • It takes two arguments: start and end.
            • 0 represents the CURRENT ROW.
            • Negative numbers (e.g., -1, -2) represent rows preceding (above) the current row.
            • Positive numbers (e.g., 1, 2) represent rows following (below) the current row.
            • For example, rowsBetween(-2, Window.currentRow) includes the current row and the two preceding rows.
        ▪ RANGE BETWEEN: Similar to rows between but defines the window frame based on logical offset values (e.g., time or value difference) rather than physical row counts. The video mentions that range between was not fully demonstrated due to a type mismatch error encountered during the recording