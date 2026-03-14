# ASAM_ODS_Poster_HighQSoft_2017_V2

## Page 1

HighQSoft GmbH • Black-und-Decker-Straße 17c • 65510 Idstein • Germany • www.highqsoft.com
Although this chart was created with the utmost care it cannot be guaranteed that it is completely free of errors or inconsistencies.
ASAM ODS Base Model 6.0
ASAM ODS 6.0 · Model 33
Name	
Value
no_interpolation	
0
linear_interpolation	
1
application_specific	
2
datatype_enum
Name	
Value
DT_UNKNOWN 	
0
DT_STRING	
1
DT_SHORT	
2
DT_FLOAT	
3
DT_BOOLEAN	
4
DT_BYTE	
5
DT_LONG	
6
DT_DOUBLE	
7
DT_LONGLONG	
8
DT_ID	
9
DT_DATE	
10
DT_BYTESTR	
11
Name	
Value
DT_BLOB	
12
DT_COMPLEX	
13
DT_DCOMPLEX	
14
DT_EXTERNALREFERENCE 	
28
DT_ENUM	
30
typespec_enum
Name	
Value
dt_boolean	
0
dt_byte	
1
dt_short	
2
dt_long	
3
dt_longlong	
4
ieeefloat4	
5
ieeefloat8	
6
dt_short_beo	
7
dt_long_beo	
8
dt_longlong_beo	
9
ieeefloat4_beo	
10
ieeefloat8_beo	
11
Name	
Value
dt_string	
12 
dt_bytestr	
13
dt_blob	
14
dt_boolean_flags_beo	
15 
dt_byte_flags_beo	
16
dt_string_flags_beo	
17 
dt_bytestr_beo	
18
dt_sbyte	
19
dt_sbyte_flags_beo	
20
dt_ushort	
21
dt_ushort_beo	
22
dt_ulong	
23
Name	
Value
dt_ushort	
21
dt_ushort_beo	
22
dt_ulong
23
dt_ulong_beo
24
dt_string_utf8	
25
dt_string_utf8_flags_beo	
26
dt_bit_int	
27
dt_bit_int_beo	
28
dt_bit_uint	
29
dt_bit_uint_beo	
30
dt_bit_ieeefloat	
31
dt_bit_ieeefloat_beo	
32
dt_bytestr_leo	
33
seq_rep_enum
Name	
Value
explicit	
0
implicit_constant	
1
implicit_linear	
2
implicit_saw	
3
raw_linear	
4
raw_polynomial	
 5
formula	
 6
external_component	
7
raw_linear_external	
 8
raw_polynomial_external	
9
raw_linear_calibrated	
10
raw_linear_calibrated_external	
11
raw_rational	
12
raw_rational_external	
13
Name	
Value
database	
0
external_only	
1
mixed	
2
foreign_format	
3
ao_storagetype_enum
interpolation_enum
Revision 2017
           
 Top Level Element
 optional	
INFO relations
 mandatory	
INFO relations
 optional	
FATHER_CHILD relations
 mandatory	
FATHER_CHILD relations
 attribute or relation optional
 attribute or relation mandatory
 more than one application relation may 
be derived from one base relation
 deprecated
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
ao_file_parent 
FATHER 
asam_base_entity 
	
[1:1]
ao_extcomp_values INFO_FROM 
AoExternalComponent 
	
[0:?]
ao_extcomp_flags 
INFO_FROM 
AoExternalComponent 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
ao_file_mimetype 
 
DT_STRING 
ao_location 
 
DT_STRING 
 
autogen
ao_size 
 
DT_LONGLONG 
 
autogen
ao_original_filename 
 
DT_STRING 
ao_original_filedate 
 
DT_DATE 
b a s e a t t r i b u t es
asam_base_entity
48
Others
AoFile
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parameter_set 
FATHER 
AoParameterSet 
	
[1:1]
unit 
INFO_TO 
AoUnit 
	
[0:1]
Name	
	
Datatype	
Mandatory	 Range
pvalue 
 
DT_STRING 
parameter_datatype 
 
DT_EMUM(datatype_enum)	
b a s e a t t r i b u t es
asam_base_entity
44
Others
AoParameter
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent 
FATHER 
AoLog 
	
[0:1]
children 
CHILD 
AoLog 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
date 
 
DT_DATE 
b a s e a t t r i b u t es
asam_base_entity
43
Others
AoLog 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent 
FATHER 
AoAny 
	
[0:1]
children  
CHILD 
AoAny 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
00
Others
AoAny 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parameters 
CHILD 
AoParameter 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
45
Others
AoParameterSet 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
Name	
	
Datatype	
Mandatory	 Range
ao_list 
 
DS_STRING 
 LIST [0:?]
b a s e a t t r i b u t es
asam_base_entity
49
Others
AoMimetypeMap
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
groups 
INFO_REL 
AoUnitGroup 
	
[0:?]
phys_dimension 
INFO_TO 
AoPhysicalDimension 
	
[1:1]
quantities 
INFO_FROM 
AoQuantity 
	
[0:?]
measurement_quantites INFO_FROM	
AoMeasurementQuantity	
	
[0:?]
parameters  
INFO_FROM 
AoParameter 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
factor 
 
DT_DOUBLE 
offset 
 
DT_DOUBLE 
b a s e a t t r i b u t es
asam_base_entity
13
Quantities and Units
AoUnit 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
default_unit 
INFO_TO 
AoUnit 
	
[0:1]
successors 
CHILD 
AoQuantity 
	
[0:?]
predecessor 
FATHER 
AoQuantity 
	
[0:1]
measurement_quantities INFO_FROM	
AoMeasurementQuantity	
	
[0:?]
groups 
INFO_REL 
AoQuantityGroup 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
default_rank 
 
DT_LONG 
default_dimension 
 
DS_LONG 
	
[0:?]
default_datatype 
 
DT_ENUM (datatype_enum)	
default_type_size 
 
DT_LONG 
default_mq_name 
 
DT_STRING 
b a s e a t t r i b u t es
asam_base_entity
11
Quantities and Units
AoQuantity 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
units 
INFO_FROM 
AoUnit 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
length_exp 
 
DT_LONG 
mass_exp 
 
DT_LONG 
time_exp 
 
DT_LONG 
current_exp 
 
DT_LONG 
temperature_exp 
 
DT_LONG 
molar_amount_exp 
 
DT_LONG 
luminous_intensity_exp 
 
DT_LONG 
length_exp_den 
 
DT_LONG 
mass_exp_den 
 
DT_LONG 
time_exp_den 
 
DT_LONG 
current_exp_den 
 
DT_LONG 
temperature_exp_den 
 
DT_LONG 
molar_amount_exp_den 
 
DT_LONG 
luminous_intensity_exp_den 
DT_LONG 
b a s e a t t r i b u t es
asam_base_entity
15
Quantities and Units
AoPhysicalDimension 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
units 
INFO_REL 
AoUnit 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
14
Quantities and Units
AoUnitGroup 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
quantities 
INFO_REL 
AoQuantity 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
12
Quantities and Units
AoQuantityGroup 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
submatrix 
FATHER 
AoSubmatrix 
	
[1:1
measurement_quantity INFO_TO 
AoMeasurementQuantity	
	
[1:1]
external_component CHILD 
AoExternalComponenty	
	
[0:?]
ao_describes 
INFO_TO 
AoLocalColumn 
	
[0:?]
ao_value_column 
INFO_TO 
AoLocalColumn 
	
[0:1]
Name	
	
Datatype	
Mandatory	 Range
flags 
 
DS_SHORT 
	
[0:?]
global_flag 
 
DT_SHORT 
independent 
 
DT_SHORT 
minimum 
 
DT_DOUBLE 
maximum 
 
DT_DOUBLE 
sequence_representation  
DT_ENUM 

 
 
(seq_rep_enum)
generation_parameters 
 
DS_DOUBLE 
	
[0:?]
raw_datatype 
 
DT_ENUM 

 
 
(datatype_enum)
values 
 
any datatype	
	
[0:?]
b a s e a t t r i b u t es
asam_base_entity
39
Measured Values
AoLocalColumn
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
local_column 
FATHER 
AoLocalColumn 
	
[1:1]
ao_values_file 
INFO_TO 
AoFile 
	
[0:1]
ao_flags_file 
INFO_TO 
AoFile	
	
[0:1]
ao_previews 
INFO_TO 
AoSubmatrix 
	
[0:1]
ao_lookup_table 
INFO_TO 
AoSubmatrix 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
ordinal_number 
 
DT_LONG 
component_length 
 
DT_LONG 
filename_url 
 
DT_STRING 
value_type 
 
DT_ENUM 

 
 
(typespec_enum)
start_offset 
 
DT_LONGLONG 
block_size 
 
DT_LONG 
valuesperblock 
 
DT_LONG 
value_offset 
 
DT_LONG 
flags_filename_url 
 
DT_STRING 
flags_start_offset 
 
DT_LONGLONG 
ao_bit_count 
 
DT_SHORT 
ao_bit_offset 
 
DT_SHORT	
b a s e a t t r i b u t es
asam_base_entity
40
Measured Values
AoExternalComponent
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
measurement 
FATHER 
AoMeasurement 
	
[1:1]
channel  
INFO_TO 
AoTestEquipment 
	
[0:1]
 
 
or AoTestEquipmentPart
 
 
or AoTestDevice
unit 
INFO_TO 
AoUnit 
	
[0:1]
quantity 
INFO_TO 
AoQuantity 
	
[0:1]
local_columns 
INFO_FROM 
AoLocalColumn 
	
[0:?]
is_scaled_by 
INFO_FROM 
AoMeasurementQuantity 
	
[0:?]
scales 
INFO_TO 
AoMeasurementQuantity	
	
[0:1]
Name	
	
Datatype	
Mandatory	 Range
datatype 
 
DT_ENUM 

 
 
(datatype_enum)
rank 
 
DT_LONG 
dimension 
 
DS_LONG 
	
[0:?]
type_size 
 
DT_LONG 
	
interpolation 
 
DT_ENUM 
	

 
 
(interpolation_enum)
minimum 
 
DT_DOUBLE 
	
maximum 
 
DT_DOUBLE 
	
average 
 
DT_DOUBLE 
	
standard_deviation 
 
DT_DOUBLE	
b a s e a t t r i b u t es
asam_base_entity
04
Measured Values
AoMeasurementQuantity
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
test 
FATHER 
AoTest 
	
[1:1]
 
 
or AoSubTest
units_under_test  
INFO_REL 
AoUnitUnderTest 
	
[0:?]
 
 
or AoUnitUnderTestPart
sequences  
INFO_REL 
AoTestSequence 
	
[0:?]
 
 
or AoTestSequencePart
equipments  
INFO_REL 
AoTestEquipment 
	
[0:?]
 
 
or AoTestEquipmentPart
 
 
or AoTestDevice
measurement_quantities CHILD 
AoMeasurementQuantity	
	
[0:?]
submatrices 
CHILD 
AoSubmatrix 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
measurement_begin 
 
DT_DATE 
measurement_end 
 
DT_DATE 
ao_values_accessed 
 
DT_DATE 
 
autogen
ao_values_accessed_by 
 
DT_STRING 
 
autogen
ao_values_modified 
 
DT_DATE 
 
autogen
ao_values_modified_by 
 
DT_STRING 
 
autogen
ao_storagetype 
 
DT_ENUM 
 
autogen
 
 
(ao_storagetype_enum)
ao_mea_size 
 
DT_LONGLONG 
 
autogen
b a s e a t t r i b u t es
asam_base_entity
03
Measured Values
AoMeasurement
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
measurement 
FATHER 
AoMeasurement 
	
[1:1]
local_columns 
CHILD 
AoLocalColumn 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
number_of_rows 
 
DT_LONG 
b a s e a t t r i b u t es
asam_base_entity
38
Measured Values
AoSubmatrix
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
groups 
INFO_REL 
AoUserGroup 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
password 
 
DT_STRING 
ao_disabled 
 
DT_SHORT 
b a s e a t t r i b u t es
asam_base_entity
34
Security
AoUser 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
users 
INFO_REL 
AoUser 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
superuser_flag 
 
DT_SHORT 
b a s e a t t r i b u t es
asam_base_entity
35
Security
AoUserGroup 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
environment 
FATHER 
AoEnvironment 
	
[0:1]
children 
CHILD 
AoSubTest 
	
[0:?]
 
 
or AoMeasurement
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
36
Management Data
AoTest 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent_test 
FATHER 
AoTest or 
	
[1:1]
 
 
AoSubTest
children 
CHILD 
AoSubTest 
	
[0:?]
 
 
or AoMeasurement
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
02
Management Data
AoSubTest
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
environment 
FATHER 
AoEnvironment 
	
[0:1]
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
children  
CHILD 
AoUnitUnderTestPart 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
21
Descriptive Data
AoUnitUnderTest 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
environment 
FATHER 
AoEnvironment 
	
[0:1]
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
children  
CHILD 
AoTestSequencePart 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
25
Descriptive Data
AoTestSequence 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
environment 
FATHER 
AoEnvironment 
	
[0:1]
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
measurement_quantities INFO_FROM 
AoMeasurementQuantity	
	
[0:?]
children  
CHILD 
AoTestEquipmentPart 
	
[0:?]
 
 
or AoTestDevice
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
23
Descriptive Data
AoTestEquipment 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent_unit_under_test FATHER 
AoUnitUnderTest or 
	
[1:1]
parent_unit_under_test_part 
AoUnitUnderTestPart
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
children  
CHILD 
AoUnitUnderTestPart 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
22
Descriptive Data
AoUnitUnderTestPart
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent_equipment 
FATHER 
AoTestEquipment 
	
[1:1]
parent_equipment_part 
or AoTestEquipmentPart
 
 
or AoTestDevice
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
measurement_quantities INFO_FROM 
AoMeasurementQuantity	
	
[0:?]
children  
CHILD 
AoTestDevice 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
37
Descriptive Data
AoTestDevice
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent_sequence 
FATHER 
AoTestSequence or 
	
[1:1]
parent_sequence_part 
AoTestSequencePart
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
children  
CHILD 
AoTestSequencePart 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
26
Descriptive Data
AoTestSequencePart
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
parent_equipment 
FATHER 
AoTestEquipment or 
	
[1:1]
parent_equipment_part  
AoTestEquipmentPart
measurement 
INFO_REL 
AoMeasurement 
	
[0:?]
measurement_quantities INFO_FROM 
AoMeasurementQuantity	
	
[0:?]
children  
CHILD 
AoTestEquipmentPart 
	
[0:?]
 
 
or AoTestDevice
Name	
	
Datatype	
Mandatory	 Range
b a s e a t t r i b u t es
asam_base_entity
24
Descriptive Data
AoTestEquipmentPart
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
name_mapping 
FATHER 
AoNameMap 
	
[1:1]
Name	
	
Datatype	
Mandatory	 Range
attribute_name 
 
DT_STRING 
alias_names 
 
DS_STRING 
	
[0:?]
b a s e a t t r i b u t es
asam_base_entity
47
Enviroment
AoAttributeMap
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
environment 
INFO_TO 
AoEnvironment 
	
[1:1]
attribute_mapping 
CHILD 
AoAttributeMap 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
entity_name 
 
DT_STRING 
alias_names 
 
DS_STRING 
	
[0:?]
b a s e a t t r i b u t es
asam_base_entity
46
Enviroment
AoNameMap 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
entity_mapping 
INFO_FROM 
AoNameMap 
	
[0:?]
tests 
CHILD 
AoTest 
	
[0:?]
uuts  
CHILD 
AoUnitUnderTest 
	
[0:?]
equipments  
CHILD	
AoTestEquipment 
	
[0:?]
sequences  
CHILD 
AoTestSequence 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
meaning_of_aliases 
 
DS_STRING 
	
[0:?]
max_test_level 
 
DT_LONG 
	
base_model_version 
 
DT_STRING 
 
asam33
application_model_type 
 
DT_STRING 
application_model_version 
DT_STRING 
timezone 
 
DT_STRING 
b a s e a t t r i b u t es
asam_base_entity
01
Enviroment
AoEnvironment 
b a s e referen c es
Name	
Relationship	
Referenced Element	
Mandatory	 Range
ao_file_children 
CHILD 
AoFile 
	
[0:?]
Name	
	
Datatype	
Mandatory	 Range
name 
 
DT_STRING 
id 
 
DT_LONGLONG 
 
autogen
version 
 
DT_STRING 
description 
 
DT_STRING 
version_date 
 
DT_DATE 
mime_type 
 
DT_STRING 
external_references 
 
DS_EXTERNALREFERENCE	
	
[0:?]
objecttype 
 
DT_LONGLONG 
ao_created 
 
DT_DATE 
 
autogen
ao_created_by 
 
DT_STRING 
 
autogen
ao_last_modified 
 
DT_DATE 
 
autogen
ao_last_modified_by 
 
DT_STRING 
 
autogen
b a s e a t t r i b u t es
asam_base_entity

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img1.jpeg)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img2.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img3.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img4.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img5.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img6.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img7.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img8.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img9.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img10.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img11.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img12.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img13.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img14.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img15.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img16.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img17.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img18.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img19.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img20.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img21.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img22.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img23.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img24.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img25.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img26.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img27.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img28.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img29.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img30.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img31.png)

![Image](./ASAM_ODS_Poster_HighQSoft_2017_V2_page1_img32.jpeg)


---
