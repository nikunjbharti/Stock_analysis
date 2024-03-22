Stock Analysis Project 
This Code will generate signal to Buy or Sell stock for short term and long-term investor, Trend of Stock, Golden Cross and Deth Cross


##Code begins from here 

CREATE TABLE ad1 (
	`date` date,
	`close_price` decimal(10,2),
    `MA7` decimal(10,2),
    `MA20` decimal(10,2),
    `MA50` decimal(10,2),
    `MA100` decimal(10,2),
    `row_num` integer
);
## creating the same table for other stocks
create table jsl1 like ad1;
create table tata1 like ad1;
create table z1 like ad1; 
select * from adanigreen

##calculating moving avg and inserting the value in new table 

insert into ad1 (date, close_price, MA7, MA20, MA50, MA100, row_num) 
( 
	select date, Close as close_price,
    avg(Close) over(order by date asc rows between 6 preceding and current row) as MA7,
    avg(Close) over(order by date asc rows between 19 preceding and current row) as MA20,
    avg(Close) over(order by date asc rows between 49 preceding and current row) as MA50,
    avg(Close) over(order by date asc rows between 99 preceding and current row) as MA100,
    row_number() over(order by date) as row_num
    from adanigreen
    order by Date
)
select * from ad1

select * from jsl
insert into jsl1 (date, close_price, MA7, MA20, MA50, MA100, row_num) 
( 
	select date, Close as close_price,
    avg(Close) over(order by date asc rows between 6 preceding and current row) as MA7,
    avg(Close) over(order by date asc rows between 19 preceding and current row) as MA20,
    avg(Close) over(order by date asc rows between 49 preceding and current row) as MA50,
    avg(Close) over(order by date asc rows between 99 preceding and current row) as MA100,
    row_number() over(order by date) as row_num
    from jsl
    order by Date
)

select * from jsl1

select * from tatapower

insert into tata1 (date, close_price, MA7, MA20, MA50, MA100, row_num) 
( 
	select date, Close as close_price,
    avg(Close) over(order by date asc rows between 6 preceding and current row) as MA7,
    avg(Close) over(order by date asc rows between 19 preceding and current row) as MA20,
    avg(Close) over(order by date asc rows between 49 preceding and current row) as MA50,
    avg(Close) over(order by date asc rows between 99 preceding and current row) as MA100,
    row_number() over(order by date) as row_num
    from tatapower
    order by Date
)

select * from tata1

select * from zomato
insert into z1 (date, close_price, MA7, MA20, MA50, MA100, row_num) 
( 
	select date, Close as close_price,
    avg(Close) over(order by date asc rows between 6 preceding and current row) as MA7,
    avg(Close) over(order by date asc rows between 19 preceding and current row) as MA20,
    avg(Close) over(order by date asc rows between 49 preceding and current row) as MA50,
    avg(Close) over(order by date asc rows between 99 preceding and current row) as MA100,
    row_number() over(order by date) as row_num
    from zomato
    order by Date
)
select * from z1

## deleting all the avg value 
## here I have used SET SQL_SAFE_UPDATES = 0 to disable the safe update mode
SET SQL_SAFE_UPDATES = 0
update ad1
set MA7 = null 
where row_num <6

update ad1
set MA20 = null 
where row_num <19

update ad1
set MA50 = null 
where row_num <49

update ad1
set MA100 = null 
where row_num <99

select * from ad1

update jsl1
set MA7 = null 
where row_num <6

update jsl1
set MA20 = null 
where row_num <19

update jsl1
set MA50 = null 
where row_num <49

update jsl1
set MA100 = null 
where row_num <99

select * from jsl1

update tata1
set MA7 = null 
where row_num <6

update tata1
set MA20 = null 
where row_num <19

update tata1
set MA50 = null 
where row_num <49

update tata1
set MA100 = null 
where row_num <99

select * from tata1

update z1
set MA7 = null 
where row_num <6

update z1
set MA20 = null 
where row_num <19

update z1
set MA50 = null 
where row_num <49

update z1
set MA100 = null 
where row_num <99

select * from z1
## dropping row_num, we don’t require this table anymore 
alter table ad1 drop row_num
alter table jsl1 drop row_num
alter table tata1 drop row_num
alter table z1 drop row_num
select * from z1
select * from jsl1
select * from tata1
select * from ad1

create table adanigreen_stock as
select B.date, A.Open, A.High, A.Low, B.close_price, A.Volume, B.MA7, B.MA20, B.MA50, B.MA100 from adanigreen A
left join ad1 B
on A.Date=B.date


select * from adanigreen_stock 
create view x as
select *, ROW_NUMBER() OVER (ORDER BY date) RowNumber
from adanigreen_stock

## creating a master table that will generate Buy and Sell signal for shoet term and long term, Trend, Golden Cross and Deth Cross for Adani Green 
create table final_analysis_adanigreen
select *, 
    case 
        when RowNumber < 100 then 
            (case 
                when RowNumber > 49 and close_price > MA50 then 'Uptrend for short term' 
                when RowNumber > 49 and close_price < MA50 then 'Downtrend for short term' 
                else 'NA' 
            end)
        when RowNumber > 99 and close_price > MA100 then 'Uptrend for long term'
        when RowNumber > 99 and close_price < MA100 then 'Downtrend for long term'
        when MA50 > MA100 and (lag(MA50,1) over (order by date)) < (lag(MA100,1) over (order by date)) then 'Golden cross'
        when MA50 < MA100 and (lag(MA50,1) over (order by date)) > (lag(MA100,1) over (order by date)) then 'Death cross'
        else 'Hold'
    end as trend,
    case 
        when RowNumber < 21 then 'NA' 
        when MA7 > MA20 and (lag(MA7,1) over (order by date)) < (lag(MA20,1) over (order by date)) then 'Short term Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Short term Sell'
        else 'Hold'
    end as signal_for_swing,
    case 
        when RowNumber < 50 then 'NA' 
        when MA20 > MA50 and (lag(MA20,1) over (order by date)) < (lag(MA50,1) over (order by date)) then 'Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Sell'
        else 'Hold'
    end as signal_for_investing
from x;
select * from final_analysis_adanigreen

#same for the JSL stocks

create table jsl_stock as
select B.date, A.Open, A.High, A.Low, B.close_price, A.Volume, B.MA7, B.MA20, B.MA50, B.MA100 from jsl A
left join jsl1 B
on A.Date=B.date


select * from jsl_stock 
create view y as
select *, ROW_NUMBER() OVER (ORDER BY date) RowNumber
from jsl_stock

select * from y
create table final_analysis_jsl
select *, 
    case 
        when RowNumber < 100 then 
            (case 
                when RowNumber > 49 and close_price > MA50 then 'Uptrend for short term' 
                when RowNumber > 49 and close_price < MA50 then 'Downtrend for short term' 
                else 'NA' 
            end)
        when RowNumber > 99 and close_price > MA100 then 'Uptrend for long term'
        when RowNumber > 99 and close_price < MA100 then 'Downtrend for long term'
        when MA50 > MA100 and (lag(MA50,1) over (order by date)) < (lag(MA100,1) over (order by date)) then 'Golden cross'
        when MA50 < MA100 and (lag(MA50,1) over (order by date)) > (lag(MA100,1) over (order by date)) then 'Death cross'
        else 'Hold'
    end as trend,
    case 
        when RowNumber < 21 then 'NA' 
        when MA7 > MA20 and (lag(MA7,1) over (order by date)) < (lag(MA20,1) over (order by date)) then 'Short term Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Short term Sell'
        else 'Hold'
    end as signal_for_swing,
    case 
        when RowNumber < 50 then 'NA' 
        when MA20 > MA50 and (lag(MA20,1) over (order by date)) < (lag(MA50,1) over (order by date)) then 'Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Sell'
        else 'Hold'
    end as signal_for_investing
from y;
select * from final_analysis_jsl

#same for tatapower 
create table tatapower_stock as
select B.date, A.Open, A.High, A.Low, B.close_price, A.Volume, B.MA7, B.MA20, B.MA50, B.MA100 from tatapower A
left join tata1 B
on A.Date=B.date


select * from tatapower_stock 
create view z as
select *, ROW_NUMBER() OVER (ORDER BY date) RowNumber
from tatapower_stock 

select * from z
create table final_analysis_tatapower
select *, 
    case 
        when RowNumber < 100 then 
            (case 
                when RowNumber > 49 and close_price > MA50 then 'Uptrend for short term' 
                when RowNumber > 49 and close_price < MA50 then 'Downtrend for short term' 
                else 'NA' 
            end)
        when RowNumber > 99 and close_price > MA100 then 'Uptrend for long term'
        when RowNumber > 99 and close_price < MA100 then 'Downtrend for long term'
        when MA50 > MA100 and (lag(MA50,1) over (order by date)) < (lag(MA100,1) over (order by date)) then 'Golden cross'
        when MA50 < MA100 and (lag(MA50,1) over (order by date)) > (lag(MA100,1) over (order by date)) then 'Death cross'
        else 'Hold'
    end as trend,
    case 
        when RowNumber < 21 then 'NA' 
        when MA7 > MA20 and (lag(MA7,1) over (order by date)) < (lag(MA20,1) over (order by date)) then 'Short term Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Short term Sell'
        else 'Hold'
    end as signal_for_swing,
    case 
        when RowNumber < 50 then 'NA' 
        when MA20 > MA50 and (lag(MA20,1) over (order by date)) < (lag(MA50,1) over (order by date)) then 'Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Sell'
        else 'Hold'
    end as signal_for_investing
from z;

select * from final_analysis_tatapower

#same for zomato

create table zomato_stock as
select B.date, A.Open, A.High, A.Low, B.close_price, A.Volume, B.MA7, B.MA20, B.MA50, B.MA100 from zomato A
left join z1 B
on A.Date=B.date


select * from zomato_stock 
create view p as
select *, ROW_NUMBER() OVER (ORDER BY date) RowNumber
from zomato_stock 

select * from p
create table final_analysis_zomato
select *, 
    case 
        when RowNumber < 100 then 
            (case 
                when RowNumber > 49 and close_price > MA50 then 'Uptrend for short term' 
                when RowNumber > 49 and close_price < MA50 then 'Downtrend for short term' 
                else 'NA' 
            end)
        when RowNumber > 99 and close_price > MA100 then 'Uptrend for long term'
        when RowNumber > 99 and close_price < MA100 then 'Downtrend for long term'
        when MA50 > MA100 and (lag(MA50,1) over (order by date)) < (lag(MA100,1) over (order by date)) then 'Golden cross'
        when MA50 < MA100 and (lag(MA50,1) over (order by date)) > (lag(MA100,1) over (order by date)) then 'Death cross'
        else 'Hold'
    end as trend,
    case 
        when RowNumber < 21 then 'NA' 
        when MA7 > MA20 and (lag(MA7,1) over (order by date)) < (lag(MA20,1) over (order by date)) then 'Short term Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Short term Sell'
        else 'Hold'
    end as signal_for_swing,
    case 
        when RowNumber < 50 then 'NA' 
        when MA20 > MA50 and (lag(MA20,1) over (order by date)) < (lag(MA50,1) over (order by date)) then 'Buy'
        when MA20 < MA50 and (lag(MA20,1) over (order by date)) > (lag(MA50,1) over (order by date)) then 'Sell'
        else 'Hold'
    end as signal_for_investing
from p;
select * from final_analysis_zomato

##view all the table 

select * from final_analysis_adanigreen 
select * from final_analysis_jsl
select * from final_analysis_tatapower
select * from final_analysis_zomato
