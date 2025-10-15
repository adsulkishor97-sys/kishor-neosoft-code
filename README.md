  private static Func<decimal, decimal> CompileFormula(string expression)
        {
            return (x) =>
            {
                var e = new Expression(expression);
                e.Parameters["x"] = (double)x; // NCalc works with double
                var result = e.Evaluate();
                return Convert.ToDecimal(result);
Covered code
            };
        }
