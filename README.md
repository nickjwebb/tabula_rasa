tabula_rasa
===========

Run community cookbooks from within AWS OpsWorks without clashing with the older versions in [opsworks-cookbooks](https://github.com/aws/opsworks-cookbooks). Recipes run from within Tabula Rasa will not have access to the opsworks-cookbooks (unless you include them in your repository) but they will see your Custom Stack JSON and all the Ohai settings added by the AWS OpsWorks Agent.

Version 0.1.0 has been tested with Chef 11.10, on Ubuntu 12.04, and only supports Git and SVN repositories.

Copyright &copy; 2014, Shlomo Swidler.

Licensed under the Apache 2.0 license.

Issues? https://github.com/shlomoswidler/tabula_rasa/issues

# Usage
AWS OpsWorks has several lifecycle actions: `setup`, `configure`, `deploy`, `undeploy`, and `shutdown`. In Custom layers you can specify the custom list of recipes to run for each action. With Tabula Rasa, you specify the run list in the Custom Stack JSON, as well as the repository from which to pull the Tabula Rasa cookbooks.

Tabula Rasa repositories can support Berkshelf. Berkshelf will be invoked if the Stack Settings also enable Berkshelf for the Stack's Custom Cookbook Repository.

To use Tabula Rasa:

1. Use Tabla Rasa `git://github.com/shlomoswidler/tabula_rasa.git` as the Custom Repository URL for your Layer (or for the entire Stack), or include this cookbook in your own custom cookbook repository (perhaps via Berkshelf - and be sure to pull in the opsworks-cookbooks as done by this cookbook's Berksfile).
2. Configure your Custom Stack JSON as per [the Configuration section below](#configuration), to specify the repository from which the Tabula Rasa cookbooks are retrieved, and the custom run list for each OpsWorks lifecycle action.
3. Include the `tabula_rasa` recipe in the OpsWorks Layer's Custom Chef Recipes for each lifecycle action. You can specify other recipes before and after `tabula_rasa` in the Custom Chef Recipes---Tabula Rasa will have no effect on them and they will run normally, as expected, using the custom cookbook repository you specified for this Layer.

To update the Tabula Rasa cookbooks from their repository (and possibly re-run Berkshelf, if it is configured to run) run the recipe `update_tabula_rasa_cookbooks`. Unfortunately, OpsWorks offers no hooks to allow you to seamlessly refresh the Tabula Rasa cookbooks when you perform an Update Custom Cookbooks.

# Configuration
The OpsWorks Custom Stack JSON should be used to specify the following items:

### Tabula Rasa cookbook repository
```
{ 
  "tabula_rasa": {
    "scm": {
      "type":       "git" or "svn" (s3 or other archives TBD)
      "repository": "Git or SVN URL",
      "revision":   "revision of the repo - HEAD is the default"
      "user":       "OPTIONAL - Git or SVN user"
      "password":   "OPTIONAL - Git or SVN password"
    }
  }
}
```

The repository SSH key specified in the Stack Settings will be used, if necessary, to fetch the Tabula Rasa cookbooks.

### Tabula Rasa run lists
```
{
  "tabula_rasa":
    "recipes": {
      "setup":      [ "recipe1", "cookbook::recipe2", "recipe3" ],
      "configure":  [ "recipe1", "cookbook::recipe2", "recipe3" ],
      "shutdown":   [ "recipe1" ]
    }
  }
}
```

Each entry in the `[:tabula_rasa][:recipes]` hash corresponds to the run list for that lifecycle action. These recipes will be run as the `tabula_rasa` recipe is executed during the converge stage of the Chef run. You can omit from the hash any lifecycle action that does not need a custom Tabula Rasa run list.


