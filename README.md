

```c++

//---------------------------------------------------------
void Load_aut(){

	const char f[] = "/aut.json";
	Serialprintln("загрузка: "+ String(f) );
	
	if (!SPIFFS.exists(f)){
		Serial.println("новый "+ String(f));
		File aut = SPIFFS.open(f, "w");
		aut.print("{\"file\": \"aut.json\",\n\n\
\"web_aut_login\": \"admin\",\n\
\"web_aut_pas\": \"admin\",\n\n\
\"BOTtoken\" : \"\",\n\
\"BOTtoken_ALARM\" : \"\",\n\n\
\"end\" : \"end\"}");
		aut.close();
	}

	File aut = SPIFFS.open(f, "r");
	String txt  = aut.readString();
	aut.close();

	config.web_aut_login 	= jsonExtract(txt, "web_aut_login", "admin"); 
	config.web_aut_pas 		= jsonExtract(txt, "web_aut_pas", "admin"); 
	
#if defined(f_TLGRM) 
	config.BOTtoken 		= jsonExtract(txt, "BOTtoken", "");
	config.BOTtoken_ALARM 	= jsonExtract(txt, "BOTtoken_ALARM", "");
#endif
}



//---------------------------------------------------------
int Load_a_setting(String filename){

	File r = SPIFFS.open(filename.c_str(), "r");
	String txt = (r.readString());
	Serialprintln("загрузка: "+ String(r) );
	r.close();

	GPIO_00_lastState 	= jsonExtract(txt, "GPIO_00", 2); // 2 - неизвестное
	GPIO_02_lastState 	= jsonExtract(txt, "GPIO_02", 2); 
	GPIO_04_lastState 	= jsonExtract(txt, "GPIO_04", 2); 
	GPIO_05_lastState 	= jsonExtract(txt, "GPIO_05", 2); 
	GPIO_12_lastState 	= jsonExtract(txt, "GPIO_12", 2); 
	GPIO_13_lastState 	= jsonExtract(txt, "GPIO_13", 2); 
	GPIO_14_lastState 	= jsonExtract(txt, "GPIO_14", 2); 
	GPIO_15_lastState 	= jsonExtract(txt, "GPIO_15", 2); 
	GPIO_16_lastState 	= jsonExtract(txt, "GPIO_16", 2); 
	
	GPIO_17_lastState 	= jsonExtract(txt, "GPIO_17", 2); 
	GPIO_18_lastState 	= jsonExtract(txt, "GPIO_18", 2); 
	GPIO_19_lastState 	= jsonExtract(txt, "GPIO_19", 2); 
	GPIO_21_lastState 	= jsonExtract(txt, "GPIO_21", 2); 
	GPIO_22_lastState 	= jsonExtract(txt, "GPIO_22", 2); 
	GPIO_23_lastState 	= jsonExtract(txt, "GPIO_23", 2); 
	GPIO_25_lastState 	= jsonExtract(txt, "GPIO_25", 2); 
	GPIO_26_lastState 	= jsonExtract(txt, "GPIO_26", 2); 
	GPIO_27_lastState 	= jsonExtract(txt, "GPIO_27", 2); 
	GPIO_32_lastState 	= jsonExtract(txt, "GPIO_32", 2); 
	GPIO_33_lastState 	= jsonExtract(txt, "GPIO_33", 2); 

#if defined(f_extIO) 	
	extIO1_0_lastState = jsonExtract(txt, "extIO1_0", 2); 
	extIO1_1_lastState = jsonExtract(txt, "extIO1_1", 2); 
	extIO1_2_lastState = jsonExtract(txt, "extIO1_2", 2); 
	extIO1_3_lastState = jsonExtract(txt, "extIO1_3", 2); 
	extIO1_4_lastState = jsonExtract(txt, "extIO1_4", 2); 
	extIO1_5_lastState = jsonExtract(txt, "extIO1_5", 2); 
	extIO1_6_lastState = jsonExtract(txt, "extIO1_6", 2); 
	extIO1_7_lastState = jsonExtract(txt, "extIO1_7", 2); 
#endif

#if defined(f_Timers) //#endif
	timers_on = jsonExtract(txt, "timers_on" , 0); 
 	for (byte Q = 0; Q < 10; Q++){
		String xx = jsonExtract(txt, "t"+String(Q)+"enable", "1"); 
		if (xx == ""){xx = "1";}
		memcpy(timer_enable_array[Q]	,xx.c_str() , 2);
		memcpy(timer_date_array[Q]		,jsonExtract(txt, "t"+String(Q)+"date", "").c_str() , 8 	);
 		memcpy(timer_time_array[Q]		,jsonExtract(txt, "t"+String(Q)+"time", "").c_str() , 4 	);
		memcpy(timer_command_array[Q]	,jsonExtract(txt, "t"+String(Q)+"comand", "").c_str() , 100 	);
	}	
#endif

return 1;

}

//---------------------------------------------------------
// return a sub-json struct
String jsonExtract(const String& json, const String& nameArg, const String& d){
  char next;
  int start = 0, stop = 0;
  static const size_t npos = -1;
  const String QUOTE = "\"";
  
  String name = QUOTE + nameArg + QUOTE;
	if (json.indexOf(name) == npos) {
		Serial.println("\""+nameArg + "\" : , не найден,  = " + d);
		return d;
	}
  start = json.indexOf(name) + name.length() + 1;
  next = json.charAt(start);

p1:
  if(next == ' ' || next == ':'){
    //Serial.println(".. a string");
    start = start + 1;
  next = json.charAt(start);
  goto p1;  
  }

  if(next == '\"'){
    //Serial.println(".. a string");
    start = start + 1;
    stop = json.indexOf('"', start);
  }
  else if(next == '['){
    //Serial.println(".. a list");
    int count = 1;
    int i = start;
    while(count > 0 && i++ < json.length()){
      if(json.charAt(i) == ']'){
	count--;
      }
      else if(json.charAt(i) == '['){
	count++;
      }
    }
    stop = i + 1;
  }
  else if(next == '{'){
    //Serial.println(".. a struct");
    int count = 1;
    int i = start;
    while(count > 0 && i++ < json.length()){
      if(json.charAt(i) == '}'){
	count--;
      }
      else if(json.charAt(i) == '{'){
	count++;
      }
    }
    stop = i + 1;
  }
  else if(next == '.' || next == '-' || ('0' <= next  && next <= '9')){
    //Serial.println(".. a number");
    int i = start;
    while(i++ < json.length() && (json.charAt(i) == '.' || ('0' <= json.charAt(i)  && json.charAt(i) <= '9'))){
    }
    stop = i;
  }
  
  //Serial.println(nameArg+" = "+json.substring(start, stop));
  return json.substring(start, stop);
}




//---------------------------------------------------------
int jsonExtract(const String& json, const String& nameArg, int d){
	String r = jsonExtract(json, nameArg, String(d));
	if (r == ""){return d;}else{return r.toInt();}
}

```
















jsonlib
=======
Simple JSON parsing library for arduino.

## Usage example

```c++
  //Remove white space (outside of string literals) from json String
  json = jsonRemoveWhiteSpace(json);
  
  String posStr = jsonExtract(json, "coord");          // {"lon":-77.35,"lat":38.93}
  float lat = jsonExtract(posStr, "lat").toFloat();    // 38.93
  float lon = jsonExtract(posStr, "lon").toFloat();    // -77.35
  String weather_list = jsonExtract(json, "weather");  // [{"id":500,"main":"Rain","description":"light rain","icon":"10d"}]
  Serial.print("weather_list:");
  Serial.println(weather_list);
  String weather_0 = jsonIndexList(weather_list, 0);   // {"id":500,"main":"Rain","description":"light rain","icon":"10d"}
  Serial.print("weather_0:");
  Serial.println(weather_0);
  String desc = jsonExtract(weather_0, "description"); // light rain
  Serial.printf("Location:%f,%f.  Current conditions: %s", lat, lon, desc.c_str());
```

## Reference

### jsonRemoveWhiteSpace

```c++
String jsonRemoveWhiteSpace(String json);
```

Returns a String with white space outside of string literals removed.  For instance
```c++
{"testing" : "one, two, three"} ==> {"testing":"one, two, three"}
```

### jsonIndexList

```c++
String jsonIndexList(String json, int idx);
```

Returns a String containing indexed item from provided String containing a json object.

### jsonExtract
```c++
String jsonExtract(String json, String name);
```

Returns a String containing the named item from provided String containing a json object. If name does not exist returns an empty String.
