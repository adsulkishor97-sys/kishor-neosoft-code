 public static string ReplaceAssetQuery(string overalN, string formatedStartdate, string formatedEnddate, string formatedStartdateyymmdd, string formatedEnddateyymmdd, string affiliateRequest1, GetAssetBenchmarkRequest request, string quatedpmcode, string sapIds)
 {
     var query = overalN!
                     .Replace("StartDateyyyymmdd", formatedStartdateyymmdd)
                     .Replace("EndDateyyyymmdd", formatedEnddateyymmdd)
                     .Replace("affiliateRequest", affiliateRequest1)
                     .Replace("${startDateyyyymm}", "'" + formatedStartdate + "'")
                     .Replace("${endDateyyyymm}", "'" + formatedEnddate + "'")
                     .Replace("${startDate}", "'" + formatedStartdate + "'")
                     .Replace("${endDate}", "'" + formatedEnddate + "'")

                     .Replace("${sapId}",   sapIds );
                     
     return query;
 }
  public static string ReplacePlantQuery(string overalN, string formatedStartdate, string formatedEnddate,string formatedStartdateyymmdd,string formatedEnddateyymmdd, string affiliateRequest1, GetPlantBenchmarkRequest request, string quatedpmcode, string plantIds, string templateIdlist)
 {


     var query = overalN!
                    
                     .Replace("affiliateRequest1", affiliateRequest1)
                     .Replace("StartDateyyyymmdd", formatedStartdateyymmdd)
                     .Replace("EndDateyyyymmdd", formatedEnddateyymmdd)
                     .Replace("affiliateRequest", affiliateRequest1)
                     .Replace("${plantIdList}", request.plantId)
                     .Replace("plantIdList", request.plantId)
                     .Replace("StartDateyyyymm", formatedStartdate)
                     .Replace("${startDateyyyymm}", "'" + formatedStartdate + "'")
                     .Replace("${endDateyyyymm}", "'" + formatedEnddate + "'")
                     .Replace("EndDateyyyymm", formatedEnddate)
                     .Replace("quatedpmcode", quatedpmcode)
                     .Replace("${templateId}", templateIdlist)
                     .Replace("${startDate}", "'" + formatedStartdate + "'")
                     .Replace("${endDate}", "'" + formatedEnddate + "'");


     return query;
 }
 Method has 10 parameters, which is greater than the 7 authorized.


