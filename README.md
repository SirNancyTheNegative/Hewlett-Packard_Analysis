# Hewlett-Packard and the Silver Tsunami

## Overview

### Purpose

In this project, we seek to solve a simple, yet potentially devastating problem: As time goes on, older people at Hewlett-Packard are looking to retire. The only issue with this is, Hewlett-Packard doesn't have a clue as to how many employees comprise this group, hereafter referred to as the "Silver Tsunami". It is, then, our duty to determine how many employees will be retiring, as well as determine how many presently-employed employees who aren't part of the Silver Tsunami could take part in a mentorship program in order to introduce new hires into the company.

## Analysis

### The Magnitude of the Tsunami

In order to determine what Hewlett-Packard will be missing with the advent of the Silver Tsunami, we need to determine how many employees will be leaving and what their most current titles are. In order to do the former, we simply need to merge the 	employee table with the titles table, specifically preening for employee numbers, first and last names, titles, and dates that those titles were first applied and previously applied, ordering them by employee number:

```
SELECT e.emp_no, e.first_name, e.last_name, t.title, t.from_date, t.to_date
INTO retirement_titles
FROM employees AS e
INNER JOIN titles AS t
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
ORDER BY e.emp_no ASC;
```

From this, we get a table, "retirement_titles", that looks like this:

![alt text](https://raw.githubusercontent.com/SirNancyTheNegative/Hewlett-Packard_Analysis/main/Resources/retirement_titles.png "A table containing every employee and all their job titles")

This isn't as helpful as we need, unfortunately. As can be seen, there are duplicate values in terms of employee numbers, meaning that this table isn't current, as it includes older titles as well. To rectify this, we need to write the following block:

```
-- Use Distinct with Orderby to remove duplicate rows
SELECT DISTINCT ON (emp_no) emp_no,
first_name,
last_name,
title
INTO unique_titles
FROM retirement_titles
ORDER BY emp_no ASC, to_date DESC;
```

This gives us the "unique_titles" table, which looks like:

![alt  text](https://raw.githubusercontent.com/SirNancyTheNegative/Hewlett-Packard_Analysis/main/Resources/unique_titles.png "A table containing every employee and their most recent job title")

This is much more manageable -- since the table is ordered by employee number lowest-to-highest, and our to_date is sorted newest-to-oldest, the distinction on the employee number means we will only ever see one copy of each employee number, and we will only ever see their most recent title. With this, we can get the titles that will be freed up when the employees entitled as such retire. Doing so is as simple as running the following query:

```
SELECT count(title), title
INTO retiring_titles
FROM unique_titles
GROUP BY title
ORDER BY count(title) DESC;
```

And this gives us a count of each title that will disappear as the owner of such a title retires:

![alt  text](https://raw.githubusercontent.com/SirNancyTheNegative/Hewlett-Packard_Analysis/main/Resources/retiring_titles.png "A table consisting of the counts of each job title as it appears in unique_titles")

From this we can glean a couple of facts:
* The majority of titles retiring come from Senior positions, both Engineer and Staff
* If every retiring-age employee were to leave relative to their birth year, over the course of three years the company will need to fill 90,398 positions.
* The lion's share of these positions, additionally, are either normal or senior engineers and staff; very few are managers or other kinds of leadership positions.
* Of the 90,398 positions that will need filling, only 2 are managers.

With this information, we can discern how best to tackle this issue with the other employees.

### Mentorship for Replacements

Because there would be 90,398 positions that need to be refilled and replaced, there needs to be a system in place where new hires would have to be shown systems already in place in order to ensure that mistakes happen as infrequently as possible. As such, we were asked to come up with a list of employees that could feasibly serve as mentors -- we're given the stipulation of anyone born in the year 1965. In order to determine who all would suffice as a mentor, we run the following query below:

```
select DISTINCT ON (e.emp_no) e.emp_no, e.first_name, e.last_name, e.birth_date, 
de.from_date, de.to_date, t.title
INTO mentor_eligibility
FROM employees as e
INNER JOIN dept_emp as de
ON (e.emp_no = de.emp_no)
INNER JOIN titles as t
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31') AND (de.to_date > '9000-01-01')
ORDER BY e.emp_no ASC, de.to_date DESC;
```

And this produces a "mentor_eligibility" table that consists of each current employee born in 1965 that would be eligible for this mentorship program.

From this, we learn some more facts:
* There are 1,549 current employees born in 1965.
* There are more regular-level staff and engineers than there are senior staff and engineer.
* There are no managers in this group.
* The majority of the titles are normal- and senior-level staff and engineers.

## Summary

### Division of Mentorship

The Silver Tsunami will bring about an opening of 90,398 job titles, and whether this is split over three years or all at once remains to be seen. The fact of the matter is, however, that these titles are divided up as such:

![alt text](https://raw.githubusercontent.com/SirNancyTheNegative/Hewlett-Packard_Analysis/main/Resources/retiring_titles.png "A table consisting of the counts of each job title as it appears in unique_titles")

As can be seen, there are thousands or even tens of thousands of each of these titles, save for managers, of which there are two. Similarly, the counts of titles that we gain from our mentor_eligibility table are as such:

![alt text](https://raw.githubusercontent.com/SirNancyTheNegative/Hewlett-Packard_Analysis/main/Resources/counts_of_mentors.png "A table consisting of the counts of each job title as it appears in mentor_eligibility")

If we limit our queries to only 1965, we don't permit much in terms of getting mentors for the new staff. Even if we average the number of titles leaving by month and assign mentors for each month, we still, for example, have one senior engineer mentor for every three incoming senior engineers that are new to the position. Suffice it to say, there won't be nearly enough mentors for incoming newcomers if we limit it to only employees who were born in 1965. If it's opened up to a sufficiently large range before and after that year, on the other hand, Hewlett-Packard could have mentors to spare, especially if taken on a month-by-month basis. This would also permit other discrepancies to be filled, especially when those two managers retire and must be replaced as well.
