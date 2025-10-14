private static async Task<List<ConvertedKpiItemPlantDetails>> GenerateConvertedMaintenancePlantReport(KpiFormula kpiFormula, KpiDetail report)
{
    return await Task.Run(() =>
    {
        var converted = new List<ConvertedKpiItemPlantDetails>();
        foreach (var item in report.plants ?? new List<Plant>())
        {
            var yValue = kpiFormula.formula!(item.actual); // Convert % to decimal for formula
            if (yValue < 0)
                yValue = 0;
            converted.Add(new ConvertedKpiItemPlantDetails
            {
                // kpi = item.kpi,
                plant = item.plantName,
                actualY = yValue,
                target = "100%",
                min = 0,
                max = 2
            });
        }
        return converted;
    });
}
 public class KpiFormula
 {
     public string? name { get; set; }
     public Func<decimal, decimal>? formula { get; set; }
     public decimal target { get; set; }
 }
  public class KpiDetail
 {
     public string? kpi { get; set; }
     public decimal average { get; set; }
     public decimal target { get; set; }
     public string? bestAffiliateName { get; set; }
     public decimal bestAffiliate { get; set; }
     public List<Affiliate>? affiliates { get; set; }
     public List<Plant>? plants { get; set; }
 }
 public class Plant
{

    public string? plantName { get; set; }  // affiliate
    public decimal actual { get; set; }
    public decimal target { get; set; }
    public int state { get; set; }
}
 public class ConvertedKpiItemPlantDetails
 {
     public string? kpi { get; set; }
     public string? plant { get; set; }
     public decimal actualY { get; set; }
     public string? target { get; set; }
     public decimal min { get; set; }
     public decimal max { get; set; }
 }
