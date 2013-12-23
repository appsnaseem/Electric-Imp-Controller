Electric-Imp-Controller
=======================


This App help users to control the Electric Imp pins by the android app "Electric Imp Controller".

In order to use this App:

	1) Go to https://plan.electricimp.com
	2) Login and make sure that your electric imp appear in the left side
	3) Create a new project(model) with the below code(Agent and Device)
	4) Upload(Build and Run) the project(model) into your Imp device.


Configure your android App(Agent id/User id). When you run the app device you requested to enter you "agent id"

- Agent id:

	https://agent.electricimp.com/[Electric Imp agent id]
	You can get your agent id from Electric imp planner.
	The "Electric Imp agent id" is case sensitive.

- User id:

	"User id" is just value to inspect and identify user actions in the logger.
	The default "user id" is "123456".
	The "User id" is case sensitive.
	Warning: When you change the user id, please  make sure you change the Users Array() in the Agent file also.
	

	
	
Device Code
-----------

	imp.configure("Smart Home", [], []);
	 
	// create a global variabled called dev, 
	dev1 <- hardware.pin1;
	dev2 <- hardware.pin2;
	dev3 <- hardware.pin5;
	dev4 <- hardware.pin7;
	dev5 <- hardware.pin8;
	dev6 <- hardware.pin9;
	 
	// configure dev to be a digital output
	dev1.configure(DIGITAL_OUT);
	dev2.configure(DIGITAL_OUT);
	dev3.configure(DIGITAL_OUT);
	dev4.configure(DIGITAL_OUT);
	dev5.configure(DIGITAL_OUT);
	dev6.configure(DIGITAL_OUT);
	
	/*dev1.write(1);
	dev2.write(1);
	dev3.write(1);
	dev4.write(1);
	dev5.write(1);
	dev6.write(1);*/
	
	function setDev1(dev1State) {
	  server.log("Set dev1: " + dev1State);
	  dev1.write(dev1State);
	}
	
	function setDev2(dev2State) {
	  server.log("Set dev2: " + dev2State);
	  dev2.write(dev2State);
	}
	 
	
	
	
	function setDev3(dev3State) {
	  server.log("Set dev3: " + dev3State);
	  dev3.write(dev3State);
	
	}
	
	function setDev4(dev4State) {
	  server.log("Set dev4: " + dev4State);
	  dev4.write(dev4State);
	
	}
	
	function setDev5(dev5State) {
	    server.log("Set dev5: " + dev5State);
	    dev5.write(dev5State);
	    /*if (dev5State == 0) {
	        imp.wakeup(2.0, turnOff5);
	    }*/
	}
	
	function setDev6(dev6State) {
	    server.log("Set dev6: " + dev6State);
	    dev6.write(dev6State);
	    /*if (dev6State == 0) {
	        imp.wakeup(2.0, turnOff6);
	    }*/
	}
	
	function getStatus(notUsed) {
	    agent.send("status", [dev1.read(), dev2.read(), dev3.read()  
	                        , dev4.read(), dev5.read(), dev6.read()]);
	}
	
	function turnOff5() {
	  server.log("auto turn off: Set dev5: " + 1);
	  dev5.write(1);
	}
	
	function turnOff6() {
	  server.log("auto turn off: Set dev6: " + 1);
	  dev6.write(1);
	}
	
	 
	 // register a handler for  messages from the agent
	agent.on("dev1", setDev1);
	agent.on("dev2", setDev2);
	agent.on("dev3", setDev3);
	agent.on("dev4", setDev4);
	agent.on("dev5", setDev5);
	agent.on("dev6", setDev6);
	agent.on("status", getStatus);
	agent.on("isOnline", function (notUsed) {
	    agent.send("isOnline","")
	});



Agent Code
----------

	
	/** White list of valid users... you can change/add mutiple user and 
	 inspect actions on your device (see the logger) */
	// for example: 
	//ValidUserID <- ["user1","user2"];
	ValidUserID <- ["123456"];
	
	statDev1 <- -1;
	statDev2 <- -1; 
	statDev3 <- -1;
	statDev4 <- -1; 
	statDev5 <- -1; 
	statDev6 <- -1;
	
	STAT_ON <- 1;
	STAT_OFF <- 0;
	
	STATUS_OFFLINE <- "-1,-1,-1,-1,-1,-1";
	
	statOnline <- 0;
	
	function getDeviceStatus() {
	    return    statDev1 + "," + statDev2 + ","+ statDev3 
	        + ","+ statDev4 + ","+ statDev5 + ","+ statDev6;
	}
	
	/**Check if the givin user id is valid*/
	function isValidID(id) {
	    foreach (i,val in ValidUserID)
	    {
	        if (val == id) {
	            return true;
	        }
	    }
	    return false;
	} 
	
	function requestHandler(request, response) {
	
	
	
	
	
	
	  try {
	    
	    if ("uid" in request.query && isValidID(request.query.uid)) {
	        server.log(" @ Getting request from:" + request.query.uid);
	    } else {
	        server.log("### err: Getting invalid request"+ "###");
	        response.send(200, "..."); 
	        server.log(" @ Getting request from:" + request.query.uid);
	        return;
	    }
	      
	    if (!device.isconnected()){
	        response.send(200, STATUS_OFFLINE);        // if device is connected, set backgroundColor to green
	        server.log("### err: Getting request our device is OFFLINE." + "###");
	        return;
	    } 
	    
	    // Check if your Imp connecting by sending him message...
	    device.send("isOnline","");
	    if (statOnline == 0) {
	        server.log("### err: Device disconnected from agent" + "###");
	        statDev1 <- -1;
	        statDev2 <- -1;
	        statDev3 <- -1;
	        statDev4 <- -1;
	        statDev5 <- -1;
	        statDev6 <- -1;
	        response.send(200, getDeviceStatus());
	        return;
	    }
	    statOnline = 0;
	
	    
	    if ("status" in request.query) {
	        response.send(200, getDeviceStatus());
	        server.log("@Device status request:" + getDeviceStatus());
	        // getting pin status from Imp
	        device.send("status",""); 
	        return;
	    } else    
	        // server.log(request.query.dev1);
	    // check if the user sent dev1 as a query parameter
	    if ("dev1" in request.query) {
	      
	      // if they did, and dev1=1.. set our variable to 1
	      if (request.query.dev1 == "1" || request.query.dev1 == "0") {
	        // convert the dev1 query parameter to an integer
	        local dev1 = request.query.dev1.tointeger();
	
	        // send "dev1" message to device, and send ledState as the data
	        device.send("dev1", dev1); 
	        device.send("status",""); 
	      }
	    } else if ("dev2" in request.query) {
	      
	      // if they did, and dev2=1.. set our variable to 1
	      if (request.query.dev2 == "1" || request.query.dev2 == "0") {
	        // convert the dev2 query parameter to an integer
	        local dev2 = request.query.dev2.tointeger();
	 
	        // send "dev2" message to device, and send ledState as the data
	        device.send("dev2", dev2); 
	        device.send("status",""); 
	      }
	    
	    } else if ("dev2" in request.query) {
	      
	      // if they did, and dev2=1.. set our variable to 1
	      if (request.query.dev2 == "1" || request.query.dev2 == "0") {
	        // convert the dev2 query parameter to an integer
	        local dev2 = request.query.dev2.tointeger();
	 
	        // send "dev2" message to device, and send ledState as the data
	        device.send("dev2", dev2); 
	        device.send("status",""); 
	      }
	    
	    } else if ("dev3" in request.query) {
	      
	      // if they did, and dev3=1.. set our variable to 1
	      if (request.query.dev3 == "1" || request.query.dev3 == "0") {
	        // convert the dev2 query parameter to an integer
	        local dev3 = request.query.dev3.tointeger();
	 
	        // send "dev2" message to device, and send ledState as the data
	        device.send("dev3", dev3); 
	        device.send("status",""); 
	      }
	    
	    } else if ("dev4" in request.query) {
	      
	      // if they did, and dev3=1.. set our variable to 1
	      if (request.query.dev4 == "1" || request.query.dev4 == "0") {
	        // convert the dev2 query parameter to an integer
	        local dev4 = request.query.dev4.tointeger();
	 
	        // send "dev2" message to device, and send ledState as the data
	        device.send("dev4", dev4); 
	        device.send("status",""); 
	      }
	    
	    } else if ("dev5" in request.query) {
	      
	      // if they did, and dev3=1.. set our variable to 1
	      if (request.query.dev5 == "1" || request.query.dev5 == "0") {
	        // convert the dev2 query parameter to an integer
	        local dev5 = request.query.dev5.tointeger();
	 
	        device.send("dev5", dev5); 
	        device.send("status",""); 
	      }
	    
	    } else if ("dev6" in request.query) {
	      
	      // if they did, and dev3=1.. set our variable to 1
	      if (request.query.dev6 == "1" || request.query.dev6 == "0") {
	        // convert the dev2 query parameter to an integer
	        local dev6 = request.query.dev6.tointeger();
	 
	        device.send("dev6", dev6); 
	        device.send("status",""); 
	      }
	    
	    } else {
	        server.log("### err: dev not found ###");
	        server.log("body:" + request.body);
	    }
	    
	    // send a response back saying everything was OK.
	    response.send(200, getDeviceStatus());
	  } catch (ex) {
	      
	    server.log("### err: " + ex + "###");
	
	    response.send(500, "ERROR");
	    
	  }
	}
	
	function writeStatus(status) {
	    local res = "";
	    if ( statDev1 != status[0]) {
	        res += "[1]:" + statDev1 + "-> "+ status[0];
	    }
	    if ( statDev2 != status[1]) {
	        res += ",[2]:" + statDev2 + "-> "+ status[1];
	    }
	    if ( statDev3 != status[2]) {
	        res += ",[3]:" + statDev3 + "-> "+ status[2];
	    }
	    if ( statDev4 != status[3]) {
	        res += ",[4]:" + statDev4 + "-> "+ status[3];
	    }
	    if ( statDev5 != status[4]) {
	        res += ",[5]:" + statDev5 + "-> "+ status[4];
	    }
	    if ( statDev6 != status[5]) {
	        res += ",[6]:" + statDev6 + "-> "+ status[5];
	    }
	    if (res != "") {
	        server.log("### Device status changed: " + res);
	    }
	    
	    // server.log("### Device status :[1]:" + statDev1 + "-> "+ status[0] 
	    // + ",[2]:" + statDev2 + "-> "+ status[1]
	    // + ",[3]:" + statDev3 + "-> "+ status[2] 
	    // + ",[4]:" + statDev4 + "-> "+ status[3] 
	    // + ",[5]:" + statDev5 + "-> "+ status[4] 
	    // + ",[6]:" + statDev6 + "-> "+ status[5] );
	    
	    statDev1 = status[0];
	    statDev2 = status[1];
	    statDev3 = status[2];
	    statDev4 = status[3];
	    statDev5 = status[4];
	    statDev6 = status[5];
	        
	    // server.log("IMP STATUS: " + status[0] + "," + status[1]+ "," + status[2]
	    // + "," + status[3]+ "," + status[4]+ "," + status[5]);
	    
	    
	}
	
	device.on("status",writeStatus);
	
	 
	// register the HTTP handler
	http.onrequest(requestHandler);
	 
	device.onconnect(function() {
	    device.send("isOnline","");
	    device.send("status",""); 
	    server.log("device connected to agent");
	});
	 
	device.ondisconnect(function() {
	    server.log("device disconnected from agent");
	});
	device.on("isOnline",function (notUsed) {
	        // server.log("!!!!!!!!!!!!!!!!!!!!device connected to agent");
	    statOnline = 1;
	});




 
