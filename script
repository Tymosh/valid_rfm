#!/usr/bin/python
import pprint
import sys
 
import xml.dom.minidom
from xml.dom.minidom import Node
 
abbrMap={	"Generation":			["Gen","and"]
		, 	"Variant":				["Var","and or"]
		, 	"Revision":				["",""]
		, 	"E_Param":				["",""]
		, 	"Official_support":		["",""]
		}
		

variantsMap={}

cur_variant=0
cur_revision=0
#magical number: because of tekst nodes number of childNodes should be 5  
EXPECTED_NUMBER_OF_CHILDNODES=5

INITIAL_VALUE_FOR_REVISION=-1
INITIAL_VALUE_FOR_UNSUPPORTED_REVISION=100

class Variant:

	def __init__(self, name, last_supported_revision, first_unsupported_revision, list_of_revisions):
		self.name 						= name		
		self.last_supported_revision 	= last_supported_revision
		self.first_unsupported_revision = first_unsupported_revision
		self.list_of_revisions 			= list_of_revisions	

def prepare_expression(list, expression):	
		
	#if is_there_an_error_in_a_string(expression):	#checks if expression contains word "ERROR"
	#	return expression							#if it does, then break the recursion
		
	for node in list.childNodes:
		if (node.nodeType != 3): # if not a TEXT_NODE; reason: we have "\n" characters in the xml file
			
			if (node.nodeName == "E_Param"):
				continue
				
			if (node.nodeName == "Variant"):
				set_current_variant(node.attributes["hex"].nodeValue, node.attributes["name"].nodeValue)				
			
			if (node.nodeName == "Revision"):
				set_current_revision(node.attributes["hex"].nodeValue)
				#check if there all needed nodes (E_Param, Official_support)				
				if (len(node.childNodes) < EXPECTED_NUMBER_OF_CHILDNODES):	
					return "ERROR: Not all mandatory tags are added ["\
					+variantsMap[cur_variant].name+"::revision "+node.attributes["hex"].nodeValue+"]!"
			
			if(node.nodeName == "Official_support"):
				set_latest_supported_revision(node.attributes["value"].nodeValue)
				set_first_unsupported_revision(node.attributes["value"].nodeValue)
				if is_there_supported_revision_after_unsupported(node.attributes["value"].nodeValue):
					return "ERROR: There is supported revison after unsupported one ["\
					+variantsMap[cur_variant].name+"::revision "+str(cur_revision)+"]!"
								
			expression=prepare_expression(node, expression)		#rolling into the deep

			if is_there_an_error_in_a_string(expression):		#checks if expression contains word "ERROR"
					return expression							#if it does, then break the recursion			
		
			#after returning from recursion [Official_support,Revision] and variants (which not contain supported revisions)
			#should be ignored
			
			if	(node.nodeName == "Official_support") or\
				(node.nodeName == "Revision") or\
				(	(node.nodeName == "Variant") and\
					(variantsMap[int(node.attributes["hex"].nodeValue,16)].last_supported_revision ==\
					INITIAL_VALUE_FOR_REVISION)):				
				continue
				
			if (node.nodeName == "Generation"):
				expression="0 "+expression
						
			expression=expression+abbrMap[node.nodeName][0]+" "+node.attributes["hex"].nodeValue+" eq "
			if (node.nodeName == "Variant"):				
				variantsMap[int(node.attributes["hex"].nodeValue,16)].list_of_revisions.sort()	
				expression=expression+\
				"E 0x"+str(variantsMap[int(node.attributes["hex"].nodeValue,16)].last_supported_revision)+" le "				
			expression=expression+abbrMap[node.nodeName][1]+" "
		
	return expression
	
def is_there_an_error_in_a_string(expression):
	if len(expression) > 0:
		return (expression.split(" ")[0] == "ERROR:")
	return False
	
def is_there_a_hole_in_numeration():
	for variant in variantsMap.keys():	
		variantsMap[variant].list_of_revisions.sort()
		if set(range(variantsMap[variant].list_of_revisions[0],variantsMap[variant].list_of_revisions[-1]+1)) != set(variantsMap[variant].list_of_revisions):
			print("ERROR: Within ",variantsMap[variant].name," variant, revisions are numbered not in sequence!" )
			return True			
	return False

def set_current_variant(variant, name):
	global cur_variant
	cur_variant=int(variant, 16)
	variantsMap[cur_variant]=Variant(name,INITIAL_VALUE_FOR_REVISION,INITIAL_VALUE_FOR_UNSUPPORTED_REVISION,[])
		
def set_current_revision(revision):	
	global cur_revision	
	cur_revision=int(revision,16)
	variantsMap[cur_variant].list_of_revisions.append(cur_revision)

def set_latest_supported_revision(value):		
	if	(value == "yes"):	
		if	( variantsMap[cur_variant].last_supported_revision < cur_revision):
			variantsMap[cur_variant].last_supported_revision = cur_revision			

def set_first_unsupported_revision(value):		
	if	(value == "no"):	
		if	( variantsMap[cur_variant].first_unsupported_revision > cur_revision):
			variantsMap[cur_variant].first_unsupported_revision = cur_revision
	
def is_there_supported_revision_after_unsupported(value):
	if	(value == "yes"):	
		return	(variantsMap[cur_variant].first_unsupported_revision < cur_revision)
	return False
		
def no_errors(string):
	if is_there_an_error_in_a_string(string):	
		print(string.split("!")[0])
		return False
		
	return not is_there_a_hole_in_numeration()
		
def main():
	args=list(sys.argv)
	if len(args) != 2:
		print("Wrong number of arguments!!! Usage: python",args[0],"hwid.xml>")
		return
	FILE=args[1]	
	hwid_doc = xml.dom.minidom.parse(FILE)	
	RPN=prepare_expression(hwid_doc.childNodes[0],"")
	
	if no_errors(RPN):
		print(RPN)
		
main()
