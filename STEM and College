import re
import PyPDF2
from typing import Dict, List, Union

def analyze_requirements_from_pdf(pdf_path: str) -> Dict[str, List[Dict[str, Union[str, int]]]]:
    """
    Analyzes a PDF file for STEM major and education level requirements.
    
    Args:
        pdf_path (str): Path to the PDF file
        
    Returns:
        Dict containing found requirements and their locations
    """
    
    # Patterns for STEM requirements
    stem_patterns = [
        r"STEM majors? only",
        r"for STEM majors",
        r"must be majoring in STEM",
        r"STEM students? only",
        r"restricted to STEM majors",
        r"open to STEM majors",
        r"limited to STEM majors",
        r"STEM background required",
        r"STEM-related major"
    ]
    
    # Patterns for education level requirements
    education_patterns = [
        # University/College patterns
        r"(?:must be|current|enrolled|active)(?:\s+[a-z]+){0,3}\s+(?:university|college)\s+student",
        r"undergraduate students? only",
        r"graduate students? only",
        r"enrolled in (?:a|an) (?:university|college)",
        r"current (?:university|college) students?",
        r"pursuing (?:a|an) (?:undergraduate|graduate) degree",
        
        # High School patterns (negative cases)
        r"high school (?:student|graduate)",
        r"secondary school (?:student|graduate)",
        r"(?:recent )?high school graduate",
    ]
    
    results = {
        "stem_requirements": [],
        "education_requirements": [],
        "page_count": 0
    }
    
    try:
        # Open and read the PDF file
        with open(pdf_path, 'rb') as file:
            pdf_reader = PyPDF2.PdfReader(file)
            results["page_count"] = len(pdf_reader.pages)
            
            # Process each page
            for page_num in range(len(pdf_reader.pages)):
                page = pdf_reader.pages[page_num]
                text = page.extract_text()
                
                # Search for STEM requirements
                for pattern in stem_patterns:
                    for match in re.finditer(pattern, text, re.IGNORECASE):
                        results["stem_requirements"].append({
                            "found_text": match.group(),
                            "page_number": page_num + 1,
                            "start_pos": match.start(),
                            "end_pos": match.end(),
                            "context": text[max(0, match.start()-50):min(len(text), match.end()+50)]
                        })
                
                # Search for education requirements
                for pattern in education_patterns:
                    for match in re.finditer(pattern, text, re.IGNORECASE):
                        # Determine if it's a college/university requirement or high school
                        is_college = not any(hs_term in match.group().lower() 
                                          for hs_term in ["high school", "secondary school"])
                        
                        results["education_requirements"].append({
                            "found_text": match.group(),
                            "page_number": page_num + 1,
                            "start_pos": match.start(),
                            "end_pos": match.end(),
                            "is_college_requirement": is_college,
                            "context": text[max(0, match.start()-50):min(len(text), match.end()+50)]
                        })
                
        return results
    
    except FileNotFoundError:
        return {"error": "PDF file not found"}
    except Exception as e:
        return {"error": f"Error processing PDF: {str(e)}"}

def print_requirement_analysis(results: Dict[str, List[Dict[str, Union[str, int]]]]) -> None:
    """
    Prints the analysis results in a readable format.
    """
    if "error" in results:
        print(f"Error: {results['error']}")
        return
        
    print(f"\nAnalyzed {results['page_count']} pages\n")
    
    print("STEM Major Requirements Found:")
    if results["stem_requirements"]:
        for item in results["stem_requirements"]:
            print(f"\nPage {item['page_number']}:")
            print(f"- Found: '{item['found_text']}'")
            print(f"- Context: '...{item['context']}...'")
    else:
        print("- No STEM requirements found")
        
    print("\nEducation Level Requirements Found:")
    if results["education_requirements"]:
        for item in results["education_requirements"]:
            requirement_type = "College/University" if item["is_college_requirement"] else "High School"
            print(f"\nPage {item['page_number']} ({requirement_type}):")
            print(f"- Found: '{item['found_text']}'")
            print(f"- Context: '...{item['context']}...'")
    else:
        print("- No education level requirements found")

# Example usage
if __name__ == "__main__":
    # You would need to provide the path to your PDF file
    pdf_path = "path/to/your/document.pdf"
    results = analyze_requirements_from_pdf(pdf_path)
    print_requirement_analysis(results)
