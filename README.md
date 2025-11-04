 public async Task<KpiTooltipResponse> GetKPITooltipBySapIdAsync(KpiInputBySapIdRequest request, string startDate, string endDate)
 {
     List<KpiTooltipDetails> sqlDataList = await _assetRepository.GetKPITooltipDetailFromSqlDBAsync(request);

     List<Label> lstLabel = new List<Label>();
     
     var startDateTime = startDate != null ? DateTime.Parse(startDate, CultureInfo.InvariantCulture) : DateTime.Now;
     var endDateTime = endDate != null ? DateTime.Parse(endDate, CultureInfo.InvariantCulture) : DateTime.Now;
     //convert format as yyyy-mm-dd 2025-08-01
     var formatedStartdate = startDateTime.ToString("yyyy-MM-dd");
     var formatedEnddate = endDateTime.ToString("yyyy-MM-dd");
     //convert format as yyyymm 202508
     var startDateMonth = startDateTime.ToString("yyyyMM");
     var endDateMonth = endDateTime.ToString("yyyyMM");


     AssetToolTipQueries assetToolTipQueries = new AssetToolTipQueries();
     List<AMHDomain.Models.BigData.AssetToolTip> assetToolTips = assetToolTipQueries.assetToolTips!.ToList();

     var tasks = assetToolTips
      .Where(a => a.kpiCode == request?.kpiCode)
      .Select(async item =>
      {
          var query = item.query!
                  .Replace("${sapId}", request?.sapId)
                  .Replace("${startDate}", $"'{formatedStartdate}'")
                  .Replace("${endDate}", $"'{formatedEnddate}'")
                  .Replace("${formatedStartdate}", startDateMonth)
                  .Replace("${formatedEnddate}", endDateMonth)
                  .Replace("${StartDateyyyymmdd}", formatedStartdate)
                  .Replace("${EndDateyyyymmdd}", formatedEnddate)
                  .Replace("${StartDateyyyymm}", $"'{startDateMonth}'")
                  .Replace("${EndDateyyyymm}", $"'{endDateMonth}'");



          var result = await _assetRepository.ExecuteBigDataQueryAsync<dynamic>(query, reader =>
          {
              var dict = new Dictionary<string, object?>();
              for (int i = 0; i < reader.FieldCount; i++)
              {
                  dict[reader.GetName(i)] = reader.IsDBNull(i) ? null : reader.GetValue(i);
              }
              return dict;
          });
          if (result.Count > 0 && result != null)
          {
              var row = result[0];

              lstLabel.Add(new Label
              {
                  title = "Absolute",
                  value = row["absolute"] != null ? Convert.ToString(row["absolute"]) : "0",
                  uom = sqlDataList.FirstOrDefault()?.kpiUom
              });
              lstLabel.Add(new Label
              {
                  title = "Orders",
                  value = row["orders"] != null ? Convert.ToString(row["orders"]) : "0",
                  uom = "#"
              });
          }
          return lstLabel;
      }).ToList();
     var data = (await Task.WhenAll(tasks!)).ToList();
     var listLabels = data.SelectMany(x => x).ToList();
     var finalResponse = new KpiTooltipResponse
     {
         labels = listLabels,
         kpiTooltipDetails = new KpiTooltipDetails
         {
             description = sqlDataList.FirstOrDefault()!.description,
             kpiName = sqlDataList.FirstOrDefault()!.kpiName,
             kpiPurpose = sqlDataList.FirstOrDefault()!.kpiPurpose,
             kpiDefinition = sqlDataList.FirstOrDefault()!.kpiDefinition,
             kpiFormula = sqlDataList.FirstOrDefault()!.kpiFormula,
             kpiUom = sqlDataList.FirstOrDefault()!.kpiUom
         }
     };
     return finalResponse;
 }
 Task<List<KpiTooltipDetails>> GetKPITooltipDetailFromSqlDBAsync(KpiInputBySapIdRequest request);
  public  class KpiInputBySapIdRequest
 {
     public string? sapId { get; set; }
     public string? kpiCode { get; set; }
     public string? startDate { get; set; }
     public string? endDate { get; set; }
 }
  public class KpiTooltipResponse
 {
     public List<Label>? labels { get; set; }
     public KpiTooltipDetails? kpiTooltipDetails { get; set; }    
 }
