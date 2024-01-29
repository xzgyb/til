# 关于Rails的命名路由方法的生成

1. 当在`config/routes.rb`中定义`resources :messages`时
2. 会导致调用Rails.application.routes.add_route  
   ```ruby
   [action_dispatch/routing/route_set.rb:630]

   name_routes[name] = route if name
   ```
   这会调用`ActionDispatch::Routing::RouteSet::NamedRouteCollection#add`
   ```ruby
   [action_dispatch/routing/route_set.rb:113]

        helper = UrlHelper.create(route, route.defaults, name)
        define_url_helper @path_helpers_module, path_name, helper, PATH
        define_url_helper @url_helpers_module, url_name, helper, UNKNOWN

        @path_helpers << path_name
        @url_helpers << url_name
   ```
3. 而`ActionDispatch::Routing::RouteSet`的url_helpers会调用generate_url_helpers创建一个module, 这个module会include @path_helpers_module和@url_helpers_module 
```ruby
  def generate_url_helpers(supports_path)
        routes = self

        Module.new do
          extend ActiveSupport::Concern
          include UrlFor

          # Define url_for in the singleton level so one can do:
          # Rails.application.routes.url_helpers.url_for(args)
          proxy_class = Class.new do
            include UrlFor
            include routes.named_routes.path_helpers_module
            include routes.named_routes.url_helpers_module

            attr_reader :_routes

            def initialize(routes)
              @_routes = routes
            end

            def optimize_routes_generation?
              @_routes.optimize_routes_generation?
            end
          end

          @_proxy = proxy_class.new(routes)

          class << self
            def url_for(options)
              @_proxy.url_for(options)
            end

            def full_url_for(options)
              @_proxy.full_url_for(options)
            end

            def route_for(name, *args)
              @_proxy.route_for(name, *args)
            end

            def optimize_routes_generation?
              @_proxy.optimize_routes_generation?
            end

            def polymorphic_url(record_or_hash_or_array, options = {})
              @_proxy.polymorphic_url(record_or_hash_or_array, options)
            end

            def polymorphic_path(record_or_hash_or_array, options = {})
              @_proxy.polymorphic_path(record_or_hash_or_array, options)
            end

            def _routes; @_proxy._routes; end
            def url_options; {}; end
          end

          url_helpers = routes.named_routes.url_helpers_module

          # Make named_routes available in the module singleton
          # as well, so one can do:
          # Rails.application.routes.url_helpers.posts_path
          extend url_helpers

          # Any class that includes this module will get all
          # named routes...
          include url_helpers
```

4. [action_controller/railite.rb:71]
```ruby
      ActiveSupport.on_load(:action_controller) do
        include app.routes.mounted_helpers
        extend ::AbstractController::Railties::RoutesHelpers.with(app.routes)
        extend ::ActionController::Railties::Helpers
```
而这个RoutesHelpers.with[abstract_controller/railties/routes_helpers.rb:8]
```ruby
    def self.with(routes, include_path_helpers = true)
        Module.new do
          define_method(:inherited) do |klass|
            super(klass)

            if namespace = klass.module_parents.detect { |m| m.respond_to?(:railtie_routes_url_helpers) }
              klass.include(namespace.railtie_routes_url_helpers(include_path_helpers))
            else
              klass.include(routes.url_helpers(include_path_helpers))
            end
          end
        end
    end
```
给ActionController::Base定义了inherited方法，使得每个controller都会去include 
Rails.application.routes.url_helpers产生的module, 这样在每个controller的action中，都可以
用resources定义产生的路由方法了.  
```ruby
 if namespace = klass.module_parents.detect { |m| m.respond_to?(:railtie_routes_url_helpers) }
              klass.include(namespace.railtie_routes_url_helpers(include_path_helpers))
```
这段代码则支持了engine中定义的route方法调用.  
其中railtie_routes_url_helpers则是由Rails::Engine的isolate_namespace方法定义的.

5. Rails.application.routes.mounted_helpers则定义了main_app和engine名字的方法代理，通过在
```ruby
      ActiveSupport.on_load(:action_controller) do
        include app.routes.mounted_helpers
        extend ::AbstractController::Railties::RoutesHelpers.with(app.routes)
        extend ::ActionController::Railties::Helpers
```
在ActionController::Base中include这个mounted_helpers, 则支持了main_app或enginge方法名称.
这个main_app是个ActionDispatch::Routing::RoutesProxy，作为一个app.routes.url_helpers的代理，使得可以调用main_app.messages_path.

6. 在ActionView的创建时
```ruby
[action_view/rendering.rb:59]
  def build_view_context_class(klass, supports_path, routes, helpers)
        if inherit_view_context_class?
          return superclass.view_context_class
        end

        Class.new(klass) do
          if routes
            include routes.url_helpers(supports_path)
            include routes.mounted_helpers
          end

          if helpers
            include helpers
          end
        end
      end
```
创建一个ActionView::Base的类，并且include url_herlps和mounted_helpers，这使得在view的各个html.erb中可以调用route定义的方法.
  
