# GRIB2Tools
Tools for processing GRIB2 files

GRIB2Tools is a library for processing GRIB2 files. GRIB2 files are usually used for storing and distributing data from weather forecasts. The library reads and decodes the meta data of the GRIB file and puts it into Java objects that are easily accessible. Furthermore, the library offers functionality for random access as well as streamed (i.e sequential) access to the data of the file.

In both cases, random as well as streamd access, an object of type <code>GribFile</code> represents the GRIB2 file. This object contains all meta data of the GRIB2 file and provides access to the data of the GRIB2 file. An object of type <code>InputStream</code> is required, which delivers the data to the <code>GribFile</code>. The <code>InputStream</code> can be obtained from any source, for example from a file on the local PC, from a resource on a FTP server or from any URL. 

<h2>Creating <code>GribFile</code> objects</h2>
A <code>GribFile</code> object for random access to the data of the GRIB2 file is created and filled with data by using the following lines: 
<p>
<code>		
    RandomAccessGribFile gribFile = new RandomAccessGribFile(datasetid, url);<br>
    gribFile.importFromStream(inputstream, 0);
</code>.
</p>
Note that the <code>RandomAccessGribFile</code> loads the complete GRIB2 file into memory, which may be a limitation, depending on the number and size of GRIB2 files you want to process and the available memory. On the other hand, the <code>RandomAccessGribFile</code> offers arbitrary and location specific random access to the data contained in the GRIB2 file using the following function
<p>
<code>    
    double longitude = ...   // in degrees<br>
    double latitude = ...    // in degrees<br>
    float val = gribFile.getValueAt(grididx, GribFile.degToUnits(latitude), GribFile.degToUnits(longiude));
</code>,
</p>
where longitude and latitude are the coordinates of the position whose data you want to obtain. Longitude and latitude can give an arbitrary position, if the position does not match a grid point, the value of the closest grid point is returned. If you need smoother data, the function
<p>
<code>    
    double longitude = ...   // in degrees<br>
    double latitude = ...    // in degrees<br>
    float val = gribFile.interpolateValueAt(grididx, GribFile.degToUnits(latitude), GribFile.degToUnits(longiude));
</code>
</p>
returns a two dimensional linear interpolated value at the position identified by longitude and latitude.

If the <code>RandomAccessGribFile</code> cannot be used due to memory constraints, a <code>StreamedGribFile</code> can be used alternatively. A <code>RandomAccessGribFile</code> is instantiated and prepared for data access with the following lines:
<p>
<code>
	StreamedGribFile gribFile = new StreamedGribFile(datasetid, url);<br>
	gribFile.prepareImportFromStream(openFile(inputstream, 0);
</code>.
</p>
These lines will load the full meta data of the GRIB2 file into memory and put it into Java classes, but it will not load the data section of the GRIB file. Instead, the data section will be read as a stream, which allows accessing the data of the GRIB2 file sequentially one by one using the function
<p>
<code>
    gribFile.float val = nextValue();
</code>.
</p>
Note, that since the data is streamed, no random access of the data for a specific location is possible. Instead, the first position has to be read from the meta data.

The second parameter of the function <code>prepareImportFromStream</code> allows to define how many GRIB2 file structures should be skipped before data is accessed. This may be useful if there are several GRIB2 file structures contained in a single file or data stream.

The parameters <code>datasetid</code> and <code>url</code> are for identification of the GRIB2 file that is represented by the <code>GribFile</code> object and do not have any further effect on the behaviour of the class.

<h2> Accessing the meta data of a GRIB2 file</h2>
The meta data of a GRIB2 file is contained in so-called templates:
<ul>
	<li>the Grid Definition Template defines the grid of the data,</li>
	<li>the Product Definition Template defines the product,</li>
	<li>the Data Representation Template defines the representation of the data.</li>
</ul>
Each of these templates exist in different variants, which are identified by the Template Numbers. In order to obtain the correct template, the Template Number has to be considered. For each of the templates, uses the scheme illustrated below to obtain the correct template:
<p>
<code>
	GridDefinitionTemplate3x gridDefinition = null;<br>
	GribSection3 section3 = gribFile.getSection3();<br>	
	if (section3.gridDefinitionTemplateNumber == 0)<br>
		gridDefinition = (GridDefinitionTemplate30)section3.gridDefinitionTemplate;<br>
	else {<br>
		log.warning("Grid Definition Template Number 3." + section3.gridDefinitionTemplateNumber + " not implemented.");<br>
		// ...<br>
	}<br>
<br>
	ProductDefinitionTemplate4x productDefinition = null;<br>
	GribSection4 section4 = gribFile.getSection4();<br>
	if (section4.productDefinitionTemplateNumber == 0)<br>
		productDefinition = ((ProductDefinitionTemplate40)section4.productDefinitionTemplate;<br>
    	else if (section4.productDefinitionTemplateNumber == 8)<br>
		productDefinition = (ProductDefinitionTemplate48)section4.productDefinitionTemplate;<br>
    	else {<br>
		log.severe("Product Definition Template Number 4." + section4.productDefinitionTemplateNumber + " not implemented.");<br>
		// ...<br>
	}<br>
<br>
	DataRepresentationTemplate5x dataRepresentation = null;<br>
	GribSection5 section5 = gribFile.getSection5(gridcnt);<br>
	if (section5.dataRepresentationTemplateNumber == 0)<br>
		dataRepresentation = ((DataRepresentationTemplate50)section5.dataRepresentationTemplate;<br>
	else {<br>
		log.severe("Data Representation Template Number 5." + section5.dataRepresentationTemplateNumber + " not implemented.");<br>
		// ...<br>
	}<br>
</code>
</p>
From the templates, the meta data of the GRIB2 file can be accessed directly as the individual data fields of the templates.

Note that the library does not fully cover the GRIB2 specification. If you feel that there are certain templates missing that should be supported by the library, please contact me or extend the lib on your own and send me a pull request.
