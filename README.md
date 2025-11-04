public class KpiNumeratorDenominatorAffiliateDistribution
{
    public int affiliates { get; set; }
    public decimal overallNumerator { get; set; }
    public decimal overallDenominator { get; set; }
    public decimal criticalNumerator { get; set; }
    public decimal criticalDenominator { get; set; }
    public int kpiid { get; set; }
    public decimal actual { get; set; }
    public string? affiliatename { get; set; }
    public int? plantid { get; set; }
    public string? sap_id { get; set; }
    public string? template { get; set; }
    public string? tidnr { get; set; }
    public string? pltxt { get; set; }
}
 public class GroupedData
 {
     public string? affiliateId { get; set; }
     public string? name { get; set; }
     public decimal overall { get; set; }
     public decimal critical { get; set; }
     public int overallState { get; set; } = 0;
     public int criticalState { get; set; } = 0;
     public long? overallTarget { get; set; } = null;
     public string? plantId { get; set; }
     public string? plantName { get; set; }

     public decimal? overallTargetMin { get; set; } = null;

     public decimal? overallTargetMax { get; set; } = null;

     public decimal? criticalTargetMin { get; set; }
     public decimal? criticalTargetMax { get; set; }
     public string? criticalTargetDisplay { get; set; } = null;
     public decimal criticalTarget { get; set; }

 }
 public class KpiNumDenAffiliateDistributionResult
{
    public int kpiid { get; set; }
    public int affiliates { get; set; }
    public string? affiliatename { get; set; }
    public int? plant { get; set; }
    public string? plantname { get; set; }
    public decimal overallNum { get; set; }
    public decimal criticalNum { get; set; }
    public decimal overallDen { get; set; }
    public decimal criticalDen { get; set; }
    public decimal overall { get; set; }
    public decimal critical { get; set; }
    public int overallState { get; set; } = 0;
    public int criticalState { get; set; } = 0;
    public long? overallTarget { get; set; } = null;
    public string? sapId { get; set; }
    public string? template { get; set; }
    public string? tidnr { get; set; }
    public string? pltxt { get; set; }
}
