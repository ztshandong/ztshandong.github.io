# LINQ lamdba
```c#
using System.Collections.Generic;
using System.Data;
using System.Linq;

namespace LINQDEMO
{
    class Program
    {
        static void Main(string[] args)
        {
            var dt1 = Test1.DataTable1.AsEnumerable();
            var per1 = Test1.People1;
            var per2 = Test1.People2;

            var tmp = from x1 in dt1
                      join x2 in per1 on x1.Field<int>("ID") equals x2.ID
                      select new Person() { ID = x1.Field<int>("ID"), Name = x1.Field<string>("Name") };
            var tmo1 = dt1.SelectMany(c => per1, (a, b) => new { a, b }).Where(@t => @t.a.Field<int>("ID") == @t.b.ID).Select(@t => new { nid = @t.b.ID, nname = @t.b.Name, @t });
            var tmp2 = per1.Where((a, index) => a.ID < index);
            var tmp3 = dt1.Select(dtr1 => new { ID = dtr1.Field<int>("ID"), Name = dtr1.Field<string>("Name") });
            var tmp4 = per1.Select(x => new { ID = x.ID, Name = x.Name });
            var tmp5 = tmp4.Union(tmp3);
            var tmp6 = tmp4.Intersect(tmp3);
            var tmp7 = tmp4.Except(tmp3);
            var tmp8 = Enumerable.Range(1, 10);
            var tmp9 = Enumerable.Repeat(1, 5);
            var tmp10 = per1.Select(x => x.ID).ToArray();
            var tmp11 = tmp10.Sum(x => x);
            var tmp12 = tmp10.Sum(x => x * 2);
            var tmp13 = tmp4.Concat(tmp3);
            var tmp14 = tmp13.GroupBy(x => x.ID).Select(g => new { gID = g.Key, gName = g });
            var tmp15 = tmp13.OrderBy(x => x.ID).ThenByDescending(x => x.Name);
            var tmp16 = dt1.SelectMany(x => per1, (a, b) => new { a, b }).Select(t => { t.a.Name = t.b.Name; return t;});
            var tmp17 = dt1.SelectMany(c => per1, (a, b) => new { a, b }).Where(@t => @t.a.Field<int>("ID") == @t.b.ID && @t.a.Field<string>("Name") == @t.b.Name).Select(@t => new { nid = @t.b.ID, nname = @t.b.Name, val = @t });
            var tmp18 = dt1.SelectMany(c => per1, (a, b) => new { a, b }).Where(@t => @t.a.Field<int>("ID") == @t.b.ID && @t.a.Field<string>("Name") == @t.b.Name).Select(@t => new Person() { ID = @t.b.ID, Name = @t.b.Name });
            int i18 = tmp18.Count();
            var tmp181 = tmp18.FirstOrDefault();
            var tmp19 = tmp18.Union(per2);
            var tmp20 = per1.Join(per2, x => x.ID, y => y.ID, (a, b) => new { a, b });
            var tmp21 = per1.GroupJoin(per2, x => x.ID, y => y.ID, (a, b) => new { a, b });
            var tmp22 = per1.GroupJoin(per2, x => x.ID, y => y.ID, (a, b) => new { a, b }).SelectMany(@t => @t.b.DefaultIfEmpty(), (@t, c) => new { @t, c }).Select(@t => new { per11 = @t.t.a, per22 = @t.c == null ? "noper2" : @t.c.ID.ToString() + @t.c.Name });

            var tmp23 = per1.GroupJoin(per2, x => x.ID, y => y.ID, (a, b) => new { a, b }).SelectMany(@t => @t.b.DefaultIfEmpty(), (@t, c) => new { @t, c }).Where(@t => @t.c != null).Select(@t => new { per11 = @t.t.a, per22 = @t.c });
            var tmp24 = per1.GroupJoin(per2, x => x.ID, y => y.ID, (a, b) => new { a, b }).SelectMany(@t => @t.b.DefaultIfEmpty(), (@t, c) => new { @t, c }).Where(@t => @t.c != null).Select(@t => new { per11 = @t.t.a, per22 = @t.t.b }).ToList().Distinct();
            Dictionary<DataRow, Person> tmp25 = dt1.Join(per1, x => x.Field<int>("ID"), y => y.ID, (a, b) => new { a, b }).Where(t => t.a.Field<string>("Name") == t.b.Name).ToDictionary(x => x.a, y => y.b);

            //foreach (var item in tmp14)
            //{
            //    foreach (var item1 in item.gName)
            //    {

            //    }
            //}
        }
    }

    public class Person
    {
        public int ID { get; set; }
        public string Name { get; set; }
    }

    public class Test1
    {

        static DataTable dataTable1 = new DataTable();

        public static DataTable DataTable1
        {
            get
            {
                return dataTable1;
            }
        }
        static List<Person> people1 = new List<Person>();

        public static List<Person> People1
        {
            get
            {
                return people1;
            }
        }
        static List<Person> people2 = new List<Person>();

        public static List<Person> People2
        {
            get
            {
                return people2;
            }
        }
        static Person person1 = new Person();
        static Person person2 = new Person();
        static Person person3 = new Person();

        static Person person4 = new Person();
        static Person person5 = new Person();
        static Person person6 = new Person();
        static Test1()
        {
            person1.ID = 1;
            person1.Name = "zhangsan1";

            person2.ID = 2;
            person2.Name = "lisi1";

            person3.ID = 3;
            person3.Name = "zhaoliu1";


            person4.ID = 1;
            person4.Name = "lisi2";

            person5.ID = 2;
            person5.Name = "zhangsan2";

            person6.ID = 1;
            person6.Name = "zhangsan2";

            people1.Add(person1);
            people1.Add(person2);
            people1.Add(person3);

            people2.Add(person4);
            people2.Add(person5);
            people2.Add(person6);

            dataTable1.Columns.Add("ID", typeof(int));
            dataTable1.Columns.Add("Name", typeof(string));

            DataRow dataRow1 = dataTable1.NewRow();
            dataRow1[0] = 1;
            dataRow1[1] = "zhangsan1";
            dataTable1.Rows.Add(dataRow1);

            DataRow dataRow2 = dataTable1.NewRow();
            dataRow2[0] = 2;
            dataRow2[1] = "lisi";
            dataTable1.Rows.Add(dataRow2);

            DataRow dataRow3 = dataTable1.NewRow();
            dataRow3[0] = 3;
            dataRow3[1] = "wangwu";
            dataTable1.Rows.Add(dataRow3);

            DataRow dataRow4 = dataTable1.NewRow();
            dataRow4[0] = 4;
            dataRow4[1] = "zhaoliu";
            dataTable1.Rows.Add(dataRow4);
        }



    }
}

```