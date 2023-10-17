- 👋 Hi, I’m @lilili1014
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
lilili1014/lilili1014 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Dialog.create("Analysis of PD-associated callose");
	Dialog.addChoice("Analysis to perform", newArray("Method A","Method B"));
	Dialog.addChoice("Source Data", newArray("Active Single Image","Active Image Stack","Folder of Images"));
	Dialog.addDirectory("Data Folder","");
	
	Dialog.addMessage("Method A Parameters", 10);
	Dialog.addNumber("Peak prominence", 100);
	Dialog.addNumber("Measurement radius", 4);
	// peak promionence was discussed here: https://forum.image.sc/t/new-maxima-finder-menu-in-fiji/25504/5
	
	Dialog.addMessage("-or-", 10);
	Dialog.addMessage("Method B Parameters", 10);
	Dialog.addNumber("Rolling ball radius", 8);
	Dialog.addNumber("Mean filter radius", 2);
	Dialog.addNumber("Auto Local Threshold radius", 180);
	Dialog.addString("Analyze Particles size filter", "25-240");
	Dialog.addString("Analyze Particles circularity filter", "0.5-1");
Dialog.show();

method = Dialog.getChoice();
data = Dialog.getChoice();
folder = Dialog.getString();

prominence = Dialog.getNumber();
mRadius = Dialog.getNumber();

rollingRadius = Dialog.getNumber();
meanRadius = Dialog.getNumber();
localRadius = Dialog.getNumber();
sizeFilter = Dialog.getString();
circFilter = Dialog.getString();

run("Set Measurements...", "area mean standard min integrated median display redirect=None decimal=3");

if (data=="Active Single Image") {
	applyMethod();
}  else if (data=="Active Image Stack") { 
	idStack = getImageID;
	n=nSlices;
	for (i=1;i<=n; i++) {
		setSlice(i);
		selectImage(idStack);
		run("Duplicate...","title=[Slice_"+i+"_"+getTitle()+"]");
		applyMethod();
		selectImage(idStack);
	}
}  else if (data=="Folder of Images") { 
	list = getFileList(folder);
	for (file=0; file<list.length; file++) {
		showProgress(file+1, list.length);
		open(folder+File.separator+list[file]);
		applyMethod();
		close("*");
	}
}
exit();

function applyMethod() {
	if (method=="Method A") doMethodA(); 
	else doMethodB();
}

function doMethodA() {
	print ("Applying Method A on "+getTitle);
	roiManager('reset');
	run("Grays");
	run("Select None");
	run("Find Maxima...", "prominence="+prominence+" output=[Point Selection]");
	getSelectionCoordinates(x,y);
	for (i=0;i<x.length;i++) {
		makeOval(x[i]-mRadius,y[i]-mRadius,mRadius*2,mRadius*2);
		run("Measure");
		roiManager('add');
	}
	roiManager("Show All with labels");
	if (!(data.startsWith("Active Single Image"))) close(); // closes duplicated slice image
}

function doMethodB() {
	print ("Applying Method B on "+getTitle);
	run("Subtract Background...", "rolling="+rollingRadius); 
	original_file_name = getTitle;
	duplicated_file_name = "Copy_of_"+original_file_name;
	run("Duplicate...", "title=[&duplicated_file_name]");
	run("Mean...", "radius="+mRadius);
	if (bitDepth>8) run ("8-bit");
	run("Auto Local Threshold...", "method=Bernsen radius="+localRadius+" parameter_1=0 parameter_2=0 white");
	setOption("BlackBackground", false);
	run("Set Measurements...", "area mean min integrated display redirect=[&original_file_name] decimal = 2");
	addToManager ="";
	if ((data.startsWith("Active Single Image"))) addToManager ="add";
	run("Analyze Particles...", "size="+sizeFilter+" pixel circularity="+circFilter+" show=[Bare Outlines] display exclude "+addToManager);	
	close(); // closes particles outlines image
	close(); // closes binary mask image
	roiManager("Show All with labels");
	if (!(data.startsWith("Active Single Image"))) {
	close(); // closes duplicated slice image
	}
}
