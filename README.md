[Fact]
public void CompileFormula_ShouldReturnCompiledFunction_ThatEvaluatesExpressionCorrectly()
{
    // Arrange
    var fixture = new Fixture();

    // Generate a random positive multiplier (no hardcoded values)
    var multiplier = fixture.Create<decimal>();
    var expression = $"x * {multiplier}";

    // Use reflection to get the private static method
    var method = typeof(PerformanceSummaryServices)
        .GetMethod("CompileFormula", BindingFlags.NonPublic | BindingFlags.Static);

    Assert.NotNull(method); // sanity check

    // Act
    var func = (Func<decimal, decimal>)method!.Invoke(null, new object[] { expression })!;
    
    // Execute the returned function with a random decimal
    var inputValue = fixture.Create<decimal>();
    var result = func(inputValue);

    // Assert
    Assert.IsType<decimal>(result);
    Assert.Equal(inputValue * multiplier, result);
    Assert.True(result >= 0 || result <= 0); // valid numeric output
}
