import java.util.*;
import java.io.*;

public class J {

import time
import re
try:
    import pyperclip
except ImportError:
    print("Error: Please install pyperclip first by running 'pip install pyperclip'")
    exit()

def generate_java_main(leetcode_code):
    # Regex to extract the main method's signature
    match = re.search(r'public\s+([a-zA-Z\[\]<>, \?]+)\s+(\w+)\s*\((.*?)\)', leetcode_code)
    
    if not match:
        return None
        
    return_type = match.group(1).strip()
    method_name = match.group(2).strip()
    args_raw = match.group(3).strip()
    
    arguments = []
    if args_raw:
        # Splitting arguments by comma
        for arg in args_raw.split(','):
            parts = arg.strip().split()
            if len(parts) >= 2:
                arg_name = parts[-1]
                arg_type = " ".join(parts[:-1])
                arguments.append((arg_type, arg_name))
                
    # Build the full Java file string
    java = "import java.util.*;\nimport java.io.*;\n\n"
    java += "public class J {\n\n"
    
    # Extract the user's logic out of LeetCode's 'class Solution' wrapper
    if "class Solution" in leetcode_code:
        inner_code = re.sub(r'class\s+Solution\s*\{', '', leetcode_code)
        inner_code = inner_code.rsplit('}', 1)[0]
        java += inner_code
    else:
        java += "    " + leetcode_code.replace("\n", "\n    ") + "\n"
        
    if not java.endswith("\n"):
        java += "\n"
        
    java += "\n    public static void main(String[] args) {\n"
    java += "        Scanner sc = new Scanner(System.in);\n"
    java += "        J solution = new J();\n\n"
    
    call_args = []
    for arg_type, arg_name in arguments:
        call_args.append(arg_name)
        if arg_type == "int":
            java += f"        int {arg_name} = sc.nextInt();\n"
        elif arg_type == "String":
            java += f"        String {arg_name} = sc.next();\n"
        elif arg_type == "double":
            java += f"        double {arg_name} = sc.nextDouble();\n"
        elif arg_type == "int[]":
            java += f"        // TODO: Read array inputs\n"
            java += f"        int[] {arg_name} = new int[]{{}};\n"
        else:
            java += f"        // TODO: Map custom inputs for {arg_type} {arg_name}\n"
            
    args_string = ", ".join(call_args)
    java += f"\n        {return_type} result = solution.{method_name}({args_string});\n\n"
    
    # Format the print statement properly based on array or primitive
    if "[]" in return_type:
        java += "        System.out.println(Arrays.toString(result));\n"
    else:
        java += "        System.out.println(result);\n"
        
    java += "    }\n

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        J solution = new J();

        // TODO: Map custom inputs for String[] args

        static void result = solution.main(args);

        System.out.println(result);
    }
}
