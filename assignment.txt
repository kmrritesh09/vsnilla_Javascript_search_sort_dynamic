var displayProducts = (function() {
	var productList;
	var showElementsCount = 20;
	var totalItems = 0;
	var selectedMenuItem = 1;
	var productDetails;

	var attachEvents = function() {
		document.getElementById('item-select').addEventListener('click', changeProductList);
		document.getElementsByClassName('products')[0].addEventListener('click', showSelectedProductDetails);
		document.getElementById('search').addEventListener('keydown', searchProducts);
		document.getElementById('sortItems').addEventListener('change', sortProducts);
	}

	var showSelectedProductDetails = function(event){
		//console.log(event);
		//debugger;
		//console.log(event.target);
		//console.log(event.target.ParentElement);

		var targetElement = event.target;
		while(targetElement.className != 'product' && targetElement.className != 'product active1'){
			targetElement = targetElement.parentNode;
		}
		changeProductClass(targetElement);

		console.log(targetElement);

		populateProductDetails(targetElement.getAttribute('data-id'));
	}

	var populateProductDetails = function(id){
		console.log(id);
		var url = "https://app.dataweave.com/v6/app/retailer/bundle_overview/?&api_key=38430b87ac715c5858b7de91fb90b3f7&base_view=all_products&bundle_id=" + id;
		getData(url).then(function(result) {
			productDetails = result.data;
			console.log(productDetails);

			var productWrapper = document.getElementsByClassName('mainDetails')[0], productData = '', tableData = '';

			productData = "<div class='productDetails'><p>" + productDetails.stock + "</p><p class='boldText bluetext'>" + productDetails.bundle_name + "</p><p>" + 
			productDetails.sku + "</p><div class='image'>" + "<img width='200' height='300' src='" + productDetails.thumbnail + "'/></div>";

			if(productDetails.your_price == "NA")
				productDetails.your_price = "-";

			if(productDetails.lowest_price_value == "NA")
				productDetails.lowest_price_value = "-";

			if(productDetails.highest_price_value == "NA")
				productDetails.highest_price_value = "-";

			var tableData = "<table class='detailsTable'><tr><td>YOUR PRICE <br/>" + productDetails.your_price + "</td></tr><tr><td>LOWEST PRICE </br>" + productDetails.lowest_price_value + "</td></tr><tr><td>HIGHEST PRICE </br>" + productDetails.highest_price_value + "</td></tr></table>";

			productData = productData + tableData + "</div>";

			productWrapper.innerHTML = productData;
		});

		
	}

	var changeProductClass = function(targetElement){
		var elems = document.querySelectorAll(".active1");
		[].forEach.call(elems, function(el) {
			el.classList.remove("active1");
		});

		targetElement.className = "product active1";
	}

	var sortProducts = function(event){
		console.log(event.target.value);
		if(event.target.value == "Price"){
			productList.sort(function(a,b){
				if(a.available_price == "NA"){
					a.available_price = 0;
				}
				if(b.available_price == "NA"){
					b.available_price = 0;
				}
				return b.available_price-a.available_price;
			});
		}
		if(event.target.value == "Discount"){
			productList.sort(function(a,b){
				if(a.discount == "NA"){
					a.discount = 0;
				}
				if(b.discount == "NA"){
					b.discount = 0;
				}
				return b.discount-a.discount;
			});
		}
		if(event.target.value == "Increase"){
			productList.sort(function(a,b){
				if(a.price_opportunity_increase_by_percentage == "NA"){
					a.price_opportunity_increase_by_percentage = 0;
				}
				if(b.price_opportunity_increase_by_percentage == "NA"){
					b.price_opportunity_increase_by_percentage = 0;
				}
				return b.price_opportunity_increase_by_percentage-a.price_opportunity_increase_by_percentage;
			});
		}
		if(event.target.value == "Decrease"){
			productList.sort(function(a,b){
				if(a.not_lowest_decrease_by_percentage == "NA"){
					a.not_lowest_decrease_by_percentage = 0;
				}
				if(b.not_lowest_decrease_by_percentage == "NA"){
					b.not_lowest_decrease_by_percentage = 0;
				}
				return b.not_lowest_decrease_by_percentage-a.not_lowest_decrease_by_percentage;
			});
		}
		populateItems();
	}

	var searchProducts = function(e){
		if (e.keyCode === 13) {  //checks whether the pressed key is "Enter"
	        var text = e.target.value;
	    	showElementsCount = 20;
	    	var url;
	    	if(text != ''){
	    		if(selectedMenuItem == 1){
	    			url = 'https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=all_products&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22' + text + '%22}&api_key=38430b87ac715c5858b7de91fb90b3f7';
	    		}
	    		else{
    				url = 'https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=increase_opportunity&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22' + text + '%22}&api_key=38430b87ac715c5858b7de91fb90b3f7';
	    		}

	    	}
	    	else
	    	{
    			if(selectedMenuItem == 1){
	    			url = 'https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=all_products&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22%22}&api_key=38430b87ac715c5858b7de91fb90b3f7';
	    		}
	    		else{
    				url = 'https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=increase_opportunity&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22%22}&api_key=38430b87ac715c5858b7de91fb90b3f7';
	    		}
	    	}
    		getData(url).then(function(result) {
				productList = result.data;
				totalItems = result.count;
				console.log(result.count);
				populateItems();
				if(totalItems > 0)
					populateProductDetails(result.data[0].bundle_id);
			});
	    }
	}

	var changeProductList = function(event) {
		showElementsCount = 20;
		selectedMenuItem = 1;
		document.getElementById('sortItems').value = "";
		var url = 'https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=all_products&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22%22}&api_key=38430b87ac715c5858b7de91fb90b3f7';
		//console.log(event);
		//console.log(event.target);
		if(event.target.textContent == "Margin gain opportunities18"){
			selectedMenuItem = 2;
			url = 'https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=increase_opportunity&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22%22}&api_key=38430b87ac715c5858b7de91fb90b3f7';
		}

		getData(url).then(function(result) {
			productList = result.data;
			totalItems = result.count;
			document.getElementById('search').placeholder = "Search from " + totalItems + " product(s)";
			console.log(result.count);
			populateItems();
			if(totalItems > 0)
				populateProductDetails(result.data[0].bundle_id);
		});

		changeMenuClass(event);
		
	}

	var changeMenuClass = function(e){
		var elems = document.querySelectorAll(".active");
		[].forEach.call(elems, function(el) {
			el.classList.remove("active");
		});
		e.target.className = "item active";
	}

	var populateItems = function(){
		var itemWrapper = document.getElementsByClassName('products')[0], listItem = '', remainingItems;
		if(totalItems < showElementsCount){
			showElementsCount = totalItems;
			remainingItems = 0;
		} 
		else{
			remainingItems = totalItems - showElementsCount;
		}
		for(var i = 0; i < (showElementsCount); i++) {
			if(productList[i].is_valid == 0){
				listItem += "<div data-id = '" + productList[i].bundle_id + "' class='product'>" + "<div class = 'text'>" + "<p class='boldText'>" + productList[i].bundle_name + 
				"<p class='greyText'>" + productList[i].sku + "<br/> <p class='bluetext'>Product not available </p></p></p></div>" + "<img width='100' height='100' src='" + productList[i].thumbnail + "'/>" + "</div>";
			}
			else if(productList[i].stock == "out of stock"){
				listItem += "<div data-id = '" + productList[i].bundle_id + "' class='product'>" + "<div class = 'text'>" + "<p class='boldText'>" + productList[i].bundle_name + 
				"<p class='greyText'>" + productList[i].sku + "<br/> <p class='bluetext'>Out of stock from <span class='redText'>" + productList[i].out_of_stock_seed_days + "</span> days</p></p></p></div>" + "<img width='100' height='100' src='" + productList[i].thumbnail + "'/>" + "</div>";
			}
			else{
				listItem += "<div data-id = '" + productList[i].bundle_id + "' class='product'>" + "<div class = 'text'>" + "<p class='bluetext'>&#x20b9;" + productList[i].available_price + "</p><p class='boldText'>" + productList[i].bundle_name + 
				"<p class='greyText'>" + productList[i].sku + "<br/> <p class='bluetext'>Increase upto <span class='redText'>" + productList[i].price_opportunity_increase_by + "</span> (<span class='redText'>" + productList[i].price_opportunity_increase_by_percentage + "%</span>) </p>" + 
				"<p  class='bluetext'>Opportunity exists from last <span class='redText'>" + productList[i].price_opportunity_days + "</span> days</p>" + "</p></p></div>" + "<img width='100' height='100' src='" + productList[i].thumbnail + "'/>" + "</div>";
			}
    	}

    	listItem = listItem + "<div class='showMore'>" + remainingItems + " product(s) more</div>"

  		itemWrapper.innerHTML = listItem;

  		document.getElementsByClassName('product')[0].className = "product active1";
	}

	var getData = function(url) {
		return fetch(url)
			.then(function(resp) {
				return resp.json();
			})
	}

	var init= function() {
		productList = getData('https://app.dataweave.com/v6/app/retailer/bundles/?&base_view=all_products&start=0&limit=20&sort_on=&sort_by=&filters={%22search%22:%22%22}&api_key=38430b87ac715c5858b7de91fb90b3f7').then(function(result) {
			productList = result.data;
			totalItems = result.count;
			console.log(result.count);
			//console.log(productList[0]);
			populateItems();
			if(totalItems > 0)
				populateProductDetails(result.data[0].bundle_id);
		});
		
		attachEvents();
		
	}
	return {
		init:init
	};

})();

displayProducts.init();