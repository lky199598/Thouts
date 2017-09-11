import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Comparator;
import java.util.Date;
import java.util.HashMap;
import java.util.Scanner;
import java.util.TreeSet;

public class ArenaManager {
	final private String invalid = new String("Error: the booking is invalid!");
	final private String success = new String("Success: the booking is accepted!");
	final private String conflicts = new String(
			"Error: the booking conflicts with Existing bookings!");
	final private String cancelled = new String(
			"Error: the booking being cancelled does not exist!");

	// 储存周天内的时间和对应的收费
	private HashMap<Integer, Integer> time1 = new HashMap<Integer, Integer>();
	// 储存周末的时间和对应的收费
	private HashMap<Integer, Integer> time2 = new HashMap<Integer, Integer>();

	// 订单信息
	private TreeSet<String> bookRecord = new TreeSet<String>();

	private TreeSet<String> fieldA = new TreeSet<String>();
	private TreeSet<String> fieldB = new TreeSet<String>();
	private TreeSet<String> fieldC = new TreeSet<String>();
	private TreeSet<String> fieldD = new TreeSet<String>();

	//HashMap存储球场收费标准
	public void charge(){
		for (int i = 9; i < 12; i++)
			time1.put(i, 30);

		for (int i = 12; i < 18; i++)
			time1.put(i, 50);

		for (int i = 18; i < 20; i++)
			time1.put(i, 80);

		for (int i = 20; i < 22; i++)
			time1.put(i, 60);

		for (int i = 9; i < 12; i++)
			time2.put(i, 40);

		for (int i = 12; i < 18; i++)
			time2.put(i, 50);

		for (int i = 18; i < 22; i++)
			time2.put(i, 60);
	}

	
	//订单的的输入
	public void inputBooking() {
		String result;

		Scanner in = new Scanner(System.in);
		
		System.out.println("您想输入几个订单（预订或取消预订）");
		int num = in.nextInt();
		String cancelEnter = in.nextLine();
		
		for(int i = 0; i < num; i++) {
	
			String input = in.nextLine();
						
			result = errorHandle(input);
		
			//成功预订，加入集合
			if (result.equals(success))
				bookRecord.add(input);
			System.out.println(result);

		}
	}

	public String errorHandle(String input) {

		String str[] = input.split(" ");
		
		if (str.length < 4)
		  return invalid;
		
		// 预订与已有预订冲突，会报错
		if (str.length == 4)
			for (String s : bookRecord) {				
				String[] book = s.split(" ");
				
				if(book.length == 4)
				if (book[3].equals(str[3])) {
					int t1 = Integer.parseInt(book[2].substring(0, 2));
					int t2 = Integer.parseInt(book[2].substring(6, 8));

					int t3 = Integer.parseInt(str[2].substring(0, 2));
					int t4 = Integer.parseInt(str[2].substring(6, 8));

					//预订与已有预订时间地点冲突，会报错
					if ((t3 >= t1 && t3 < t2) || (t4 > t1 && t3 <= t2))
						return conflicts;
				}

			}

		//时间输入不合法
		if(str[2].length() != 11 )
			return invalid;
		
		//间段的起止时间必然为整小时，否则报错
		String beginTime = str[2].substring(3, 5);
		String endTime = str[2].substring(9);
		if (!beginTime.equals("00") || !endTime.equals("00"))
			return invalid;

		// 间段的起止时间等于结束时间
		String beginTime1 = str[2].substring(0, 2);
		String endTime1 = str[2].substring(6, 8);
		if (beginTime1.equals(endTime1))
			return invalid;

		
		if (str.length == 5) {
			// 取消标记只能是C，若为其他字符则报错
			if (!str[str.length - 1].equals("C")) {
				return invalid;
			}

			// 只能完整取消之前的预订，不能取消部分时间段
			String cancel = input.substring(0, input.length() - 2);
			int flag = 0;
			for (String s : bookRecord)
				if (cancel.equals(s)) {
					bookRecord.remove(cancel);

					flag = 1;
					break;
				}

			if (flag == 0)
				return cancelled;
		}
		return success;
	}

	//程序输出
	public void outputBooking(int[] count) {
		System.out.println("\n收入汇总");
		System.out.println("---");

		System.out.println("场地:A");
		for (String s : fieldA)
			System.out.println(s);
		System.out.println("小计：" + count[0] + "元\n");

		System.out.println("场地:B");

		for (String s : fieldB)
			System.out.println(s);
		System.out.println("小计：" + count[1] + "元\n");

		System.out.println("场地:C");

		for (String s : fieldC)
			System.out.println(s);
		System.out.println("小计：" + count[2] + "元\n");

		System.out.println("场地:D");

		for (String s : fieldD)
			System.out.println(s);
		System.out.println("小计：" + count[3] + "元");

		System.out.println("---");
		System.out.println("总计：" + count[4] + "元\n");
	}

	// 计算各场地的收入和总收入
	public int[] incomeSum() {

		// count[0]~count[3]为场地A, 场地B,场地C,场地D小计,count[4]为总计
		int[] count = new int[5];

		String field = null;
		String booking = null;

		for(String s : bookRecord){

			String str[] = s.split(" ");
			field = str[3];
			int sum = 0;
			booking = str[1] + " " + str[2] + " ";

			int beginTime = Integer.parseInt(str[2].substring(0, 2));
			int endTime = Integer.parseInt(str[2].substring(6, 8));

			for (int i = beginTime; i < endTime; i++)
			{
				if(getWeek(str[1]).equals("星期六")
					|| getWeek(str[1]).endsWith("星期日"))				
					    sum += time2.get(i);
				else
					sum+=time1.get(i);
			}
			

			// 取消的预定
			if (str.length == 5) {
				if (getWeek(str[1]).equals("星期六")
						|| getWeek(str[1]).endsWith("星期日"))
					sum *= 0.25;
				else
					sum *= 0.5;
				booking += "违约金 ";
			}
			booking += sum + "元";

			if (field.equals("A")) {
				count[0] += sum;
				fieldA.add(booking);
			}

			else if (field.equals("B")) {
				count[1] += sum;
				fieldB.add(booking);
			}

			else if (field.equals("C")) {
				count[2] += sum;
				fieldC.add(booking);
			}

			else {
				count[3] += sum;
				fieldD.add(booking);
			}
		}

		for (int i = 0; i < 4; i++)
			count[4] += count[i];
		return count;

	}

	// 得到星期几
	public String getWeek(String strDate) {
		SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd");
		Date date = null;
		try {
			date = sdf1.parse(strDate);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		SimpleDateFormat sdf2 = new SimpleDateFormat("EEEE");
		String week = sdf2.format(date);
		return week;
	}
	
	public static void main(String[] args) {
		ArenaManager a = new ArenaManager();
		a.charge();   //HashMap存储球场收费标准
		a.inputBooking(); //订单的的输入
		int count[] = a.incomeSum();// 计算各场地的收入和总收入
		a.outputBooking(count);		//程序输出
	}
}



