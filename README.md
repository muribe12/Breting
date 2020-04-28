# Filipa
WITH src AS ( SELECT distinct
	          a.dim_org_id, --??
	          LEFT(CAST(a.billing_date AS string),7) AS invoice_month,
	          a.revenue_actual AS credit_burned
FROM 
	  (SELECT * FROM vmc_dw_restricted.fact_credit_burn_dash WHERE LENGTH(CAST(billing_date AS String)) > 0) a
              WHERE  a.revenue_actual IS NOT NULL
            )

SELECT
	add_months(to_timestamp(sub.invoice_month,'yyyy-MM-dd'), -1) as "invoice month",
	CAST(sub.credit_burned AS BIGINT) AS "credit burned",
	SUM(sub.credit_burned) OVER ORDER BY to_timestamp(sub.invoice_month,'yyyy-MM-dd') ROWS UNBOUNDED PRECEDING  "cumulative credit burned"

--sum(sub.credit_burned) over (order by ('01-' + sub.invoice_month)::datetime rows unbounded preceding)::BIGINT "cumulative credit burned"

FROM
	(
		SELECT 
			invoice_month,
			SUM(credit_burned) AS credit_burned
		FROM 
			src 
		GROUP BY
			invoice_month
	) AS sub
ORDER BY
	add_months(('01-' + sub.invoice_month)::datetime, -1)
