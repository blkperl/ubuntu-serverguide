+---------------------+------------------------------------------------------+
| Attribute           | Description                                          |
+=====================+======================================================+
| **vendor**          | Specifies the vendor name of the storage device to   |
|                     | which the device attributes apply, for example       |
|                     | **COMPAQ**.                                          |
+---------------------+------------------------------------------------------+
| **product**         | Specifies the product name of the storage device to  |
|                     | which the device attributes apply, for example       |
|                     | **HSV110 (C)COMPAQ**.                                |
+---------------------+------------------------------------------------------+
| **revision**        | Specifies the product revision identifier of the     |
|                     | storage device.                                      |
+---------------------+------------------------------------------------------+
| **product\_blacklis | Specifies a regular expression used to blacklist     |
| t**                 | devices by product.                                  |
+---------------------+------------------------------------------------------+
| **hardware\_handler | Specifies a module that will be used to perform      |
| **                  | hardware specific actions when switching path groups |
|                     | or handling I/O errors. Possible values include:     |
|                     |                                                      |
|                     | -  **1 emc**: hardware handler for EMC storage       |
|                     |    arrays                                            |
|                     |                                                      |
|                     | -  **1 alua**: hardware handler for SCSI-3 ALUA      |
|                     |    arrays.                                           |
|                     |                                                      |
|                     | -  **1 hp\_sw**: hardware handler for Compaq/HP      |
|                     |    controllers.                                      |
|                     |                                                      |
|                     | -  **1 rdac**: hardware handler for the LSI/Engenio  |
|                     |    RDAC controllers.                                 |
|                     |                                                      |
                                                                            
+---------------------+------------------------------------------------------+

Table: Device Attributes

