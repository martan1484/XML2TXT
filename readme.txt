# XML › TXT Converter

A lightweight tool for converting structured XML data into TXT files using a flexible configuration file.

The program reads an XML file and generates one or more TXT files according to rules defined in `config.ini`.

The configuration file (config.ini) must be located in the same directory as the executable.

File config.ini must be saved in UTF-8 encoding.

---

# Features

* Convert XML data into TXT tables
* Fully configurable output structure
* Supports nested XML structures
* Table grouping with separators
* Merge multiple XML records into a single output line
* Works on Windows and Linux
* Can be compiled into a standalone executable

---

# How it works

Each section in `config.ini` generates **one TXT output file**.

Example:

```
[BusStops_Station]
```

will produce:

```
BusStops_Station.txt
```

The converter reads XML elements and formats them into TXT rows based on the configuration rules.

---

# Configuration file (config.ini)

The behavior of the converter is defined entirely in `config.ini`.

Each section describes how one TXT file should be generated.

---

# General settings

```
[General]
output_location = 1
```

Determines where the `output` folder will be created.

| Value | Description                                         |
| ----- | --------------------------------------------------- |
| 0     | output folder is created next to the input XML file |
| 1     | output folder is created next to the executable     |

---

# Standard processing

Standard mode means:

**1 XML item = 1 output row**

Example:

```
[BusStops_Station]
section = BusStops
item = BusStop
key = Station
fields = ID,"NameFull (ZoneID)","NameFull (ZoneID)"
```

Explanation:

| Parameter | Meaning                                   |
| --------- | ----------------------------------------- |
| section   | XML section (container element)           |
| item      | repeating XML element                     |
| key       | text written before "=" in the TXT output |
| fields    | template for output values                |

Example output:

```
Station=123,"Main Street (1)","Main Street (1)"
```

---

# Fields syntax

Fields are separated by commas.

Rules:

| Syntax          | Meaning                          |
| --------------- | -------------------------------- |
| ID              | XML attribute or element value   |
| ""              | constant text                    |
| empty field     | empty column                     |
| field@Item      | reference to another XML element |
| repeated fields | allowed                          |

Examples:

```
fields = ID
fields = ID,"NameFull"
fields = ID,"NameFull (ID)"
fields = SuperID@Zone,,ID@Zone
```

---

# Nested XML access (parent)

If the required XML elements are nested deeper in the structure, you can define a path using `parent`.

Example:

```
parent = Tariffs/Tariff/PriceTable/Data
```

The converter will navigate the XML hierarchy to find the correct elements.

---

# Table grouping

Sometimes XML contains multiple tables inside the same structure.
In this case you can use `group` and `separator`.

Example:

```
group = PriceTable
separator = ID@Tariffs/Tariff,Name@Tariffs/Tariff,Payment@Tariffs/Tariff/PriceTable
```

Behavior:

When the `PriceTable` element changes, the program:

1. generates a new separator line
2. writes it to the output
3. continues processing rows for that table

Example output:

```
1,City Tariff,Cash,QR
Price=1,21.00,60
Price=2,25.00,90

1,City Tariff,EMV,QR
Price=1,21.00,60
Price=2,25.00,90
```

---

# Special processing: MERGE

Merge mode allows combining multiple XML records into a single output row.

Example configuration:

```
[Zones_MainZone]

type = merge

section = Zones
item = Zone

key = MainZone
fields = SuperID@Zone,,ID@Zone;"Name@SuperZone"

group_by = SuperID
value = ID
```

Explanation:

| Parameter  | Meaning                              |
| ---------- | ------------------------------------ |
| type=merge | enables merge mode                   |
| group_by   | XML field used for grouping records  |
| value      | value collected from grouped records |

Example output:

```
MainZone=10,,101,102,103;"Central"
```

---

# Running the program

When executed, the program will:

1. ask for an XML file
2. read `config.ini`
3. generate TXT files in the `output` folder

---

# Future improvements

* command line parameters for:

  * XML input
  * output directory
  * config file
* batch processing of multiple XML files
* optional logging
* validation of configuration

