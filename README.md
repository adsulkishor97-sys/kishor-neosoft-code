foreach (var item in jsonFileGlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
{
    var queryBestAchievedEver = ReplacePlantQuery(item.bestAchievedEver!, formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd, "", request, quatedpmcode, formatPlantIds, templateIdlist);

    var bestAchievedResult = await _benchMarkRepository.ExecuteBigDataQuery<PlantBenchmarkGroupedData>(queryBestAchievedEver, reader => new PlantBenchmarkGroupedData
    {
        plantId = reader["plant_id"] != DBNull.Value ? Convert.ToInt32(reader["plant_id"]) : 0,
        bestAchievedEver = reader["best_achieved"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved"]) : 0,
        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
        actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0

    }) ?? new List<PlantBenchmarkGroupedData>();



    if(bestAchievedResult==null || bestAchievedResult.Count == 0)
    {
        return new List<PlantBenchmarkGroupedData>
        {
            new PlantBenchmarkGroupedData
            {
                plantId=0,
                bestAchievedEver=0,
                absolute=0,
                actual=0
            }
        };
    }

    groupData = bestAchievedResult;
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

