# mapbatcher_v1.0.py
# Adrian Hughes 15 January 2013
# Version 1.0
# attempt at creating a fully functional mapbatcher using arcpy.mapping
# you need to open the data driven pages toolbar in the mxd 
# open the setup screen and select layer and relevant fields
#  see ESRI help: http://resources.arcgis.com/en/help/main/10.1/index.html#//00s90000003n000000

# the elements on the layout page must exist and be named the same
# set the checkbox on legend properties to 'only show classes that are visible to map extent'
# also note that path to category images folder is currently hardcoded

# WARNING: Currently the data driven pages will zoom to the extent of the first polygon for each species
# This will cause problems if you have more than one row per species as is often the case
# In this instance you need to create a dissolved species range layer and use that for the
# data driven pages index layer. Make it invisible in the map and use the undissolved species layer in the map
# for symbology.


import arcpy, os, sys, traceback, inspect, shutil, datetime
import IUCNSP
import time

# page layout output parameters
MAXSCALE = 500000   # maximum scale, if large than this, maximum will be used
PAGEWIDTH = 2481    # pagelayout width
PAGEHEIGHT = 3509   # pagelayout height
RESOLUTION = 300    # dot per inch (dpi)

# default RL image parameters
RLX = 12.5351
RLY = 8.3406
RLW = 6.9907
RLH = 1.82

# input parameters
layername = arcpy.GetParameterAsText(0)
binomialfield = arcpy.GetParameter(1)
rlcategory_field = arcpy.GetParameterAsText(2)
citation_field = arcpy.GetParameterAsText(3)
exportfolder = arcpy.GetParameterAsText(4)

# get current map document
mxd = arcpy.mapping.MapDocument("CURRENT")

# get dataframes
dfinset = arcpy.mapping.ListDataFrames(mxd)[0]
dfspecies = arcpy.mapping.ListDataFrames(mxd)[1]


# get the right species layer
layers = arcpy.mapping.ListLayers(mxd, "", dfinset)
for each in layers:
    if each.name == layername:
        layer = each
        break
    else:
        pass

# get a unique list of species name from the species layer
namelist = IUCNSP.GetUniqueValuesFromShapefile(layer.dataSource, binomialfield)
IUCNSP.Printboth(str(namelist))

# Start mapping...
try:
    for name in namelist:
        nameasc = name.encode("utf8","ignore")
        exportpng = exportfolder + os.sep + name + '.PNG'
        query =  str(binomialfield) + "='" + str(name) + "'"
        pageNum = mxd.dataDrivenPages.getPageIDFromName(nameasc)
        
        # find the appropriate page to draw
        mxd.dataDrivenPages.currentPageID = pageNum      
        
        # make a definition query to show the given species only
        layer.definitionQuery = query        

        # create searchcursor to get species attributes
        expression = arcpy.AddFieldDelimiters(layer.dataSource, binomialfield) + " = " + nameasc
       
        with arcpy.da.SearchCursor(layer.dataSource, (rlcategory_field,citation_field), where_clause=query) as cursor:

            for row in cursor:                
                rlcategory = row[0]
                citation = row[1]              
        
        
        # set RL category image
        for elmrlimage in arcpy.mapping.ListLayoutElements(mxd,"PICTURE_ELEMENT"):
            if elmrlimage.name == "RLimage":
                elmrlimage.sourceImage = r"S:\Adrian\Mapbatcher\images\RL_" + rlcategory + ".tif"        
        
        # set species name label
        for elmname in arcpy.mapping.ListLayoutElements(mxd,"TEXT_ELEMENT"):
            if elmname.name == "SpeciesName":
                elmname.text = str(name)
                
        # set citation label
        for elmdate in arcpy.mapping.ListLayoutElements(mxd,"TEXT_ELEMENT"):
            if elmdate.name == "Citation":
                elmdate.text = "Data Source: " + citation            
        

        # set map creation date label
        for elmdate in arcpy.mapping.ListLayoutElements(mxd,"TEXT_ELEMENT"):
            if elmdate.name == "CreationDate":
                elmdate.text = "Map created on " + IUCNSP.CurrentDate

        # create dynamic legend        
        
        if dfinset.scale < MAXSCALE:
            dfinset.scale = MAXSCALE
        # refresh, which will be done automatically when this particular page is exported    
##        arcpy.RefreshActiveView()
        
    ##    print "Exporting page {0} of {1}".format(str(mxd.dataDrivenPages.currentPageID), str(mxd.dataDrivenPages.pageCount))
        IUCNSP.Printboth ("Exporting map {0} of {1}".format(str(mxd.dataDrivenPages.currentPageID), str(mxd.dataDrivenPages.pageCount)))
        arcpy.mapping.ExportToPNG(mxd, exportpng, "PAGE_LAYOUT", PAGEWIDTH, PAGEHEIGHT, RESOLUTION)
        
except arcpy.ExecuteError: 
    # Get the tool error messages 
    # 
    msgs = arcpy.GetMessages(2) 

    # Return tool error messages for use with a script tool 
    #
    arcpy.AddError(msgs) 

    # Print tool error messages for use in Python/PythonWin 
    # 
    print msgs

except:
    # Get the traceback object
    #
    tb = sys.exc_info()[2]
    tbinfo = traceback.format_tb(tb)[0]

    # Concatenate information together concerning the error into a message string
    #
    pymsg = "PYTHON ERRORS:\nTraceback info:\n" + tbinfo + "\nError Info:\n" + str(sys.exc_info()[1])
    msgs = "ArcPy ERRORS:\n" + arcpy.GetMessages(2) + "\n"

    # Return python error messages for use in script tool or Python Window
    #
    arcpy.AddError(pymsg)
    arcpy.AddError(msgs)

    # Print Python error messages for use in Python / Python Window    
    print pymsg + "\n"
    print msgs
    
#  Regardless of whether the script succeeds or not, delete    
finally:
    if each:
        del each    
    if layers:
        del layers
    if dfinset:
        del dfinset
    if dfspecies:
        del dfspecies
    if mxd:
        del mxd
