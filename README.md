zoho-purchase-Items-script-from-Zoho-Book during Invoice Creation


invoiceID = invoice.get("invoice_id");

organizationID = organization.get("organization_id");

invoice_data = zoho.books.getRecordsByID("invoices",organizationID,invoiceID);

customer_name = invoice.get("customer_name");

account_criteria = "(Account_Name:equals:" + customer_name + ")";

account_search = zoho.crm.searchRecords("Accounts",account_criteria);

purchased_products = List();

if(account_search.size() > 0)
{
	account_data = account_search.get(0);
	account_id = account_data.get("id");
	record_owner = account_data.get("Owner").get("id");
	account_type = account_data.get("Account_Type");
 
	// Process each line item
	line_items = invoice.get("line_items");
	info line_items;
	products_subform = List();
	for each  item in line_items
	{
		product_name = item.get("name");
		purchased_products.add({"Name":product_name,"Account_Name1":{"id":account_id},"Owner":{"id":record_owner},"Account_Type":account_type,"Shipping_City":account_data.get("Shipping_City"),"Shipping_Code":account_data.get("Shipping_Code"),"Shipping_Country":account_data.get("Shipping_Country"),"Shipping_State":account_data.get("Shipping_State"),"Shipping_Street":account_data.get("Shipping_Street"),"Billing_City":account_data.get("Billing_City"),"Billing_Code":account_data.get("Billing_Code"),"Billing_Country":account_data.get("Billing_Country"),"Billing_State":account_data.get("Billing_State"),"Billing_Street":account_data.get("Billing_Street"),"Serial":item.get("sku")});
		description = item.get("description");
		quantity = item.get("quantity");
		rate = item.get("rate");
		tags_list = List();
		rental = false;
  
		// You can set this based on your logic
		if(product_name.contains("Boost 1"))
		{
			tag = Map();
			tag.put("name","Boost 1");
			tags_list.add(tag);
		}
		else if(product_name.contains("Boost 2 Elite"))
		{
			tag = Map();
			if(rental)
			{
				tag.put("name","Boost 2 Elite Rental");
			}
			else
			{
				tag.put("name","Boost 2 Elite");
			}
			tags_list.add(tag);
		}
		else if(product_name.contains("Boost 2 Core"))
		{
			tag = Map();
			if(rental)
			{
				tag.put("name","Boost 2 Core Rental");
			}
			else
			{
				tag.put("name","Boost 2 Core");
			}
			tags_list.add(tag);
		}
		else if(product_name.contains("Boost 3"))
		{
			tag = Map();
			if(rental)
			{
				tag.put("name","Boost 3 Rental");
			}
			else
			{
				tag.put("name","Boost 3");
			}
			tags_list.add(tag);
		}
		else if(product_name.contains("Shorts"))
		{
			tag = Map();
			tag.put("name","Shorts");
			tags_list.add(tag);
		}

  
		// Your insert/update logic goes here, using `param`
	}
	product_search_criteria = "(Product_Name:equals:" + product_name + ")";
	product_search = zoho.crm.searchRecords("Products",product_search_criteria);
	if(product_search.size() > 0)
	{
		product_id = product_search.get(0).get("id");
		purchased_unit_data = Map();
		purchased_unit_data.put("Products",{"id":product_id});
		purchased_unit_data.put("Account_Name",{"id":account_id});
		purchased_unit_data.put("Serial_Number",item.get("serial_number"));
		purchased_unit_data.put("Description",description);
		purchased_unit_data.put("Quantity",quantity);
		purchased_unit_data.put("Unit_Price_1",rate);
		purchased_unit_data.put("Record_Owner",record_owner);
		products_subform.add(purchased_unit_data);
		update_data = Map();
		update_data.put("Associated_Products",products_subform);
		update_response = zoho.crm.updateRecord("Accounts",account_id,update_data);

  
		// 		info update_response;
		param = Map();
		param.put("tags",tags_list);
		recordIDs = List();
		recordIDs.add(account_id);
		// Replace with your actual Account ID
		param.put("ids",recordIDs);
		res = invokeurl
		[
			url :"https://www.zohoapis.com/crm/v7/Accounts/actions/add_tags"
			type :POST
			parameters:param.toString()
			connection:"crm_ishtiyak_conn"
		];
		if(purchased_products.size() > 0)
		{
			info zoho.crm.bulkCreate("Installed_Base",purchased_products);
		}
	}
}
