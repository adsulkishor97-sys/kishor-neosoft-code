 private static (string, string, string, string) FormatDatesAffiliate(GetAffiliateBenchmarkRequest request)
 {
     var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
     var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

     var formattedStartdate = startDateTime.ToString("yyyyMM");
     var formattedEnddate = endDateTime.ToString("yyyyMM");
     var formattedStartdateYMD = startDateTime.ToString("yyyy-MM-dd");
     var formattedEnddateYMD = endDateTime.ToString("yyyy-MM-dd");

     return (formattedStartdate, formattedEnddate, formattedStartdateYMD, formattedEnddateYMD);
 }
 public class GetAffiliateBenchmarkRequest
{
    public string? affiliateId { get; set; }
    public string? kpiCode { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
}

 {
    affiliateId: "123, 124",
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",

}
