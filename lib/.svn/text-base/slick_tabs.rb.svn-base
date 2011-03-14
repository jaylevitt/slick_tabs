require 'RMagick'
require 'md5'

LOGGER = RAILS_DEFAULT_LOGGER
SLICK_DIR_NAME = 'slick_tabs'

class SlickError < StandardError; end

class Behavior::Base
  
  define_tags do
    tag "slicktabs_header" do |tag|
      stcm = SlickTabsContextMgr.instance
      stcm.create_context(tag.attr['class'], @request, page)
      stcm.css_header
    end
    
    tag "slicktabs" do |tag|
      slicktab = SlickTabs.new(tag, page)
      slicktab.html
    end
        
    [:normal, :hover, :current, :ancestor].each do |symbol|
      tag "slicktabs:#{symbol}" do |tag|
        sctm = SlickTabsContextMgr.instance
        sctm.define_style(symbol.to_s, tag)
      end
    end
    
    tag 'parent' do |tag|
      tag.locals.page = page.parent
      tag.expand
    end
 
  end
end

class SlickTabsContextMgr
  include ActionView::Helpers::AssetTagHelper
  include Singleton

  def create_context(klass, request, page)
    klass ||= 'slicktabs'
    
    # Create a dummy controller so image_path works
    @controller = ActionController::Base.new
    @controller.request = request

    @contexts = {} if @contexts.nil?
    @context = @contexts[klass] = {}
    @context['class'] = klass
    @context['page'] = page
    @context['styles'] = {}
    
    check_for_slick_dir
    @initialized = true
  end

  def focus_context(klass)
    return false unless @initialized
    
    klass ||= 'slicktabs'
    return false unless @contexts.key?(klass)

    # We don't actually know when we're done rendering the page, and we don't want
    # the context to carry over from page to page - it's our proof that we've rendered
    # the header.  Therefore, we declare that a context can be used only once - focusing
    # on it means deleting it.

    @context = @contexts.delete(klass)
    true
  end
  
 def blur_context
    @context = nil
  end
  
  def destroy_all_contexts
    # Useful for testing
    @contexts = {}
    @context = nil
  end

  def css_class
    @context['class']
  end

  def css_path
    image_path("#{SLICK_DIR_NAME}/#{css_class}-#{@context['page'].id}.css")
  end
  
  def css_path_on_server
    File.join(slick_dir_on_server, "#{css_class}-#{@context['page'].id}.css")
  end
  
  def css_header
    %Q|<link href="#{css_path}" media="screen" rel="stylesheet" type="text/css"/>|
  end
  
  def image_filename(label)
    style_hash = "%x" % (label + styles.to_s).hash.abs
    label[0..9].gsub(/[^A-Za-z0-9]/, '-') + "-#{style_hash}.png"
  end
  
  def tab_path(label)
    image_path("#{SLICK_DIR_NAME}/" + image_filename(label))
  end
  
  def tab_path_on_server(label)
    File.join(slick_dir_on_server, image_filename(label))
  end

  def define_style(name, tag)
    ['height', 'font_size'].each do |attr|
      unless tag.attr.include?(attr)
        raise SlickError, "the #{name} style must include a '#{attr}' attribute."
      end
    end
   
    styles = @context['styles']
    styles[name] = tag.attr
    return
  end

  def styles
    @context['styles']  
  end
  
  private
  
  def check_for_slick_dir
    @dir_path = slick_dir_on_server
    return if File.exists?(@dir_path) and File.directory?(@dir_path)
    
    LOGGER.info("Creating dir #{@dir_path}")
    Dir.mkdir(@dir_path)
  end

  def slick_dir_on_server
    File.join(RAILS_ROOT, "public", "images", SLICK_DIR_NAME)
  end
  
end

class SlickTabs
  include ActionView::Helpers::AssetTagHelper

  attr_reader :html
  
  def initialize(tag, page)
    @page = page
    
    unless tag.double?
      raise SlickError, "r:slicktabs must be used as a container tag." 
    end
    
    @sctm = SlickTabsContextMgr.instance
    unless @sctm.focus_context(tag.attr['class'])
      raise SlickError, "r:slicktabs must be preceded by r:slicktabs_header."
    end

    contents = tag.expand
    
    @css_class = @sctm.css_class
    @styles = @sctm.styles
    unless @styles['normal']
      raise SlickError, "r:slicktabs must contain a style for at least r:normal."
    end

    html = []
    @css = css_for_all_tabs
    @page_list = contents.scan(%r|a href="(.*)">(.*)</a>|)    
    @page_list.each do |page|
      one_tab = do_one_tab(page[1], page[0])
      html << "#{one_tab}\n"
    end
    
    if html.size == 1
      html[0].sub!(/<li>/, '<li class="first last">')
    elsif html.size > 1
      html[0].sub!(/<li>/, '<li class="first">')
      html[-1].sub!(/<li>/, '<li class="last">')
    end
    
    f = File.new(@sctm.css_path_on_server, "w")
    f.write @css
    f.close
    
    SlickTabsContextMgr.instance.blur_context
    @html = %Q|<ul class="#{@css_class}">#{html.to_s}</ul>|
  end

  private
    
  def do_one_tab(label, href)
    #TODO: save the max width and use that, not just the last one
    @styles.each do |name, style|
      @style = style
      parse_style
      format_label(label)
      determine_tab_width
    end
    
    @max_height = @styles.values.collect {|s| s['height']}.max

    @styles['hover'] ||= @styles['normal']
    @styles['current'] ||= @styles['normal']
    @styles['ancestor'] ||= @styles['normal']
    
    path = @sctm.tab_path_on_server(label)
    create_tab_image(path) unless File.exists?(path) 
    @css << css_for_tab(label) << css_for_image(label)

    a_tag = %Q|<a id="#{css_id(label)}" #{a_class(href)}href="#{href}">|
    formatted_label = @styles['normal']['formatted_label']
    html = %Q|<li>#{a_tag}<span></span>#{formatted_label}</a></li>|
    return html
  end

  def parse_style
    ['height', 'font_size', 
     'padding_left', 'padding_right', 'padding_bottom'].each do |attr|
      @style[attr] = (@style[attr] || 0).to_i
    end
    
    @style['background'] ||= 'white'
    @style['color'] ||= 'black'
    @style['blur'] = (@style['blur'] || 0.90).to_f
  end
  
  def format_label(label)
    formatted_label = label.dup
    formatted_label.sub!(/ /, "\n") if @style['two_line']
    formatted_label.upcase! if @style['uppercase']
    
    @style['formatted_label'] = formatted_label
  end
  
  def determine_tab_width
    # To get the type metrics, we can't use the tab we actually draw - the type 
    # would be twice as big as it should be!  
    tab= Magick::Image.new(1000, @style['height'])
    text = Magick::Draw.new
    text.gravity = Magick::SouthWestGravity
    text.pointsize = @style['font_size']
    text.font = File.join(RAILS_ROOT, "public", "images", "fonts", "FuturaStd-Condensed.otf")
    # TODO: add options for font, gravity
    metrics = text.get_multiline_type_metrics(tab, @style['formatted_label'])
    @width = metrics.width.to_i + @style['padding_left'] + @style['padding_right']
    #TODO: Determine size of biggest style, don't assume they're all the same
  end
  
  def create_tab_image(path)
    tab = {}
    @styles.each do |name, style|
      @style = style
      tab[name] = draw_tab
    end

    big = Magick::Image.new(@width * 2, @max_height * 2)
    @styles.each do |name, style|
      big.composite!(tab[name], style_gravity(name), Magick::OverCompositeOp)
    end

    big.write(path)
  end

  def draw_tab
    # Draw at 200% scale so the font hinting works better, then reduce it
    bg = @style['background']
    tab = Magick::Image.new(2000, @style['height'] * 2) do
      self.background_color = bg
    end
    
    text = Magick::Draw.new
    text.gravity = Magick::SouthWestGravity
    text.pointsize = @style['font_size']
    text.density = "144x144"
    text.fill = @style['color']
    text.font = File.join(RAILS_ROOT, "public", "images", "fonts", "FuturaStd-Condensed.otf")
    text.annotate(tab, 0, 0, @style['padding_left'] * 2, @style['padding_bottom'] * 2, @style['formatted_label'])
    
    tab = tab.resize(1000, @style['height'], Magick::LanczosFilter, @style['blur'])
    tab.crop!(Magick::SouthWestGravity, @width, @style['height'])
    return tab
  end
  
  def css_for_all_tabs
    <<-STOP.undent
      .#{@css_class} a {
        position: relative;
        overflow: hidden;
      }

      .#{@css_class} a span {
        position: absolute;
        background-position: top left;
        background-repeat: no-repeat;
        height: 100%; 
        width: 100%;
        cursor:hand;
      }

      .#{@css_class} a:hover span {
        background-position: top right;
      }
      
      .#{@css_class} a.current span {
        background-position: bottom left;
      }
      
      .#{@css_class} a.ancestor span {
        background-position: bottom right;
      }
    STOP
  end

  def css_for_tab(label)
    <<-STOP.undent
      .#{@css_class} a\##{css_id(label)} {
        height: #{@style['height']}px; 
        width: #{@width}px;
      }
      
    STOP
  end
  
  def css_for_image(label)
    # Gilder/Levin replacement
    <<-STOP.undent
      .#{@css_class} a\##{css_id(label)} span {
        background-image: url(#{path(label)});
      }
      
    STOP
  end

  def css_id(label)
    label.gsub(/\s/,'')
  end

  def style_gravity(name)  
    case name
    when 'normal'
      Magick::NorthWestGravity
    when 'hover'
      Magick::NorthEastGravity 
    when 'current'
      Magick::SouthWestGravity
    when 'ancestor'
      Magick::SouthEastGravity
    end
  end
      
  def path(label)
    SlickTabsContextMgr.instance.tab_path(label)
  end

  def a_class(href)
    target_url = remove_trailing_slash(href)
    current_url = remove_trailing_slash(@page.url)

    if target_url == current_url
      'class="current" '
    elsif Regexp.new("^#{target_url}").match(current_url)
      'class="ancestor" '
    else
      ''
    end
    
  end
  
  def remove_trailing_slash(string)
    (string =~ %r{^(.*?)/$}) ? $1 : string
  end 
end

class String
  def undent
    a = $1 if match(/\A(\s+)(.*\n)(?:\1.*\n)*\z/)
    gsub(/^#{a}/,'')
  end
end

