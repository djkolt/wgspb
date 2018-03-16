<?php
/*
 * 0 - PHP-генерация
 * 1 - XSL-генерация
 */
$type = 0;

// Создавать индекс
$createIndex = FALSE;

// Количество страниц в каждый файл
$perFile = 1000;

$oSite = Core_Entity::factory('Site')->getByAlias(Core::$url['host']);

$oSite_Alias = $oSite->getCurrentAlias();

if (is_null($oSite_Alias))
{
	?>Site hasn't had a default alias!<?php
	exit();
}

$oCore_Sitemap = new Core_Sitemap($oSite);
$oCore_Sitemap
	->createIndex($createIndex)
	->rebuildTime(0)
	->perFile($perFile);

if ($type == 0)
{
	$aSiteuserGroups = array(0, -1);
	if (Core::moduleIsActive('siteuser'))
	{
		$oSiteuser = Core_Entity::factory('Siteuser')->getCurrent();

		if ($oSiteuser)
		{
			$aSiteuser_Groups = $oSiteuser->Siteuser_Groups->findAll();
			foreach ($aSiteuser_Groups as $oSiteuser_Group)
			{
				$aSiteuserGroups[] = $oSiteuser_Group->id;
			}
		}
	}

	$oStructures = $oSite->Structures;
	$oStructures
		->queryBuilder()
		->where('active', '=', 1)
		->where('indexing', '=', 1)
		->where('siteuser_group_id', 'IN', $aSiteuserGroups)
		->orderBy('sorting')
		->orderBy('name');

	$aStructure = $oStructures->findAll();

	if (Core_Page::instance()->libParams['showInformationsystemGroups']
		|| Core_Page::instance()->libParams['showInformationsystemItems'])
	{
		$aInformationsystems = $oSite->Informationsystems->findAll();

		foreach ($aInformationsystems as $oInformationsystem)
		{
			$_Informationsystems[$oInformationsystem->structure_id] = $oInformationsystem;
		}
	}

	if (Core_Page::instance()->libParams['showShopGroups']
		|| Core_Page::instance()->libParams['showShopItems'])
	{
		$aShops = $oSite->Shops->findAll();

		foreach ($aShops as $oShop)
		{
			$_Shops[$oShop->structure_id] = $oShop;
		}
	}

	$dateTime = Core_Date::timestamp2sql(time());

	foreach ($aStructure as $oStructure)
	{
		$oCore_Sitemap->addNode('http://' . $oSite_Alias->name . $oStructure->getPath(), $oStructure->changefreq, $oStructure->priority);

		// Informationsystem
		if (Core_Page::instance()->libParams['showInformationsystemGroups'] && isset($_Informationsystems[$oStructure->id]))
		{
			$oInformationsystem = $_Informationsystems[$oStructure->id];

			$oInformationsystem_Groups = $oInformationsystem->Informationsystem_Groups;
			$oInformationsystem_Groups->queryBuilder()
				->where('informationsystem_groups.siteuser_group_id', 'IN', $aSiteuserGroups);
			$aInformationsystem_Groups = $oInformationsystem_Groups->findAll();

			$path = 'http://' . $oSite_Alias->name . $oInformationsystem->Structure->getPath();

			foreach ($aInformationsystem_Groups as $oInformationsystem_Group)
			{
				$oCore_Sitemap->addNode($path . $oInformationsystem_Group->getPath(), $oStructure->changefreq, $oStructure->priority);
			}

			// Informationsystem's items
			if (Core_Page::instance()->libParams['showInformationsystemItems'])
			{
				$oInformationsystem_Items = $oInformationsystem->Informationsystem_Items;
				$oInformationsystem_Items->queryBuilder()
					->select('informationsystem_items.*')
					->open()
					->where('informationsystem_items.start_datetime', '<', $dateTime)
					->setOr()
					->where('informationsystem_items.start_datetime', '=', '0000-00-00 00:00:00')
					->close()
					->setAnd()
					->open()
					->where('informationsystem_items.end_datetime', '>', $dateTime)
					->setOr()
					->where('informationsystem_items.end_datetime', '=', '0000-00-00 00:00:00')
					->close()
					->where('informationsystem_items.siteuser_group_id', 'IN', $aSiteuserGroups);
				$aInformationsystem_Items = $oInformationsystem_Items->findAll();

				foreach ($aInformationsystem_Items as $oInformationsystem_Item)
				{
					$oCore_Sitemap->addNode($path . $oInformationsystem_Item->getPath(), $oStructure->changefreq, $oStructure->priority);
				}
			}
		}

		// Shop
		if (Core_Page::instance()->libParams['showShopGroups'] && isset($_Shops[$oStructure->id]))
		{
			$oShop = $_Shops[$oStructure->id];

			$oShop_Groups = $oShop->Shop_Groups;
			$oShop_Groups->queryBuilder()
				->where('shop_groups.siteuser_group_id', 'IN', $aSiteuserGroups);
			$aShop_Groups = $oShop_Groups->findAll();

			$path = 'http://' . $oSite_Alias->name . $oShop->Structure->getPath();
			foreach ($aShop_Groups as $oShop_Group)
			{
				$oCore_Sitemap->addNode($path . $oShop_Group->getPath(), $oStructure->changefreq, $oStructure->priority);
			}

			// Shop's items
			if (Core_Page::instance()->libParams['showShopItems'])
			{
				$oShop_Items = $oShop->Shop_Items;
				$oShop_Items->queryBuilder()
					->select('shop_items.*')
					->open()
					->where('shop_items.start_datetime', '<', $dateTime)
					->setOr()
					->where('shop_items.start_datetime', '=', '0000-00-00 00:00:00')
					->close()
					->setAnd()
					->open()
					->where('shop_items.end_datetime', '>', $dateTime)
					->setOr()
					->where('shop_items.end_datetime', '=', '0000-00-00 00:00:00')
					->close()
					->where('shop_items.siteuser_group_id', 'IN', $aSiteuserGroups);
				$aShop_Items = $oShop_Items->findAll();

				foreach ($aShop_Items as $oShop_Item)
				{
					$oCore_Sitemap->addNode($path . $oShop_Item->getPath(), $oStructure->changefreq, $oStructure->priority);
				}
			}
		}
	}

	$oCore_Sitemap->execute();
}
else
{
	$Structure_Controller_Show = new Structure_Controller_Show(
		$oSite->showXmlAlias(TRUE)
	);

	$Structure_Controller_Show
		->xsl(
			Core_Entity::factory('Xsl')->getByName(Core_Page::instance()->libParams['xsl'])
		)
		//->parentId(0)
		// Показывать группы информационных систем в карте сайта
		->showInformationsystemGroups(Core_Page::instance()->libParams['showInformationsystemGroups'])
		// Показывать элементы информационных систем в карте сайта
		->showInformationsystemItems(Core_Page::instance()->libParams['showInformationsystemItems'])
		// Показывать группы магазина в карте сайта
		->showShopGroups(Core_Page::instance()->libParams['showShopGroups'])
		// Показывать товары магазина в карте сайта
		->showShopItems(Core_Page::instance()->libParams['showShopItems'])
		->show();
}

exit();
